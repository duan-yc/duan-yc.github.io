---
layout: post
title: "shortlink"
subtitle: "shortlink"
author: "DYC"
header-img: "img/post-bg-linux.jpg"
header-mask: 0.3
catalog: true
tags:
  - java
---

### shortlink

#### 短链接管理

###### 新增短链接

###### 短链接分表

###### 拦截器

###### 分页查询

###### 创建用户后默认分组

###### 短链接信息修改

1. 删除原始短链接记录

   t_link表是按照gid进行分表的，所以如果修改短链接的gid，则需要先删除原gid分表中的记录，再在新gid分表中插入修改后的短链接记录

2. 修改唯一索引

   考虑到修改gid需要先删再新添加，有可能新增的短链接又路由到了原来的表，就会和原来的造成索引冲突，所以新加了一个delete_time，新增的del_time和full_short_url组成唯一索引可以有效避免这种冲突

3. 迁移相关业务数据

   将短链接相关的表进行数据修改，如果涉及到分片行为，先删除原有数据再新增。如果不涉及分片行为，只需要修改对应的数据库表记录即可

4. 引入读写锁

   思考一个问题，如果短链接正在修改分组，这时有用户正在访问短链接，统计监控相关的分组还是之前的数据，是否就涉及到无法正确统计监控数据问题？

   所以这里就引入读写锁，进行统计监控的时候需要获取读锁，修改分组的时候需要获取写锁

5. 引入延迟队列

   考虑这个场景，如果用户正在修改短链接分组，因为涉及到表操作很多，我们假设可能会操作 300ms。

   这 300ms 内难道就不允许用户访问？

   所以引入延迟队列，在监控的时候，如果没办法获取读锁，就将相关数据加入延迟队列，延迟队列会在5s后再次进行更新

   ```java
   public class DelayShortLinkStatsConsumer implements InitializingBean {
   
       private final RedissonClient redissonClient;
       private final ShortLinkService shortLinkService;
   
       public void onMessage() {
           Executors.newSingleThreadExecutor(
                           runnable -> {
                               Thread thread = new Thread(runnable);
                               thread.setName("delay_short-link_stats_consumer");
                               thread.setDaemon(Boolean.TRUE);
                               return thread;
                           })
                   .execute(() -> {
                       RBlockingDeque<ShortLinkStatsRecordDTO> blockingDeque = redissonClient.getBlockingDeque(DELAY_QUEUE_STATS_KEY);
                       RDelayedQueue<ShortLinkStatsRecordDTO> delayedQueue = redissonClient.getDelayedQueue(blockingDeque);
                       for (; ; ) {
                           try {
                               ShortLinkStatsRecordDTO statsRecord = delayedQueue.poll();
                               if (statsRecord != null) {
                                   shortLinkService.shortLinkStats(null, null, statsRecord);
                                   continue;
                               }
                               LockSupport.parkUntil(500);
                           } catch (Throwable ignored) {
                           }
                       }
                   });
       }
   
       @Override
       public void afterPropertiesSet() throws Exception {
           onMessage();
       }
   }
   ```

6. 回收站删除

7. 修改相关接口

###### 短链接跳转

大致思路：根据短链接进行查表，因为是分表情况，所以要建立一个路由表，通过路由表找到长连接，然后进行跳转

这里需要考虑缓存穿透和缓存击穿问题，所以解决思路如下

1. 在redis中查询
2. 布隆过滤器查询
3. 在redis中查询是否为空（在后面如果路由表中没有的话会加入redis 下一次来这里就会进行拦住）
4. 使用分布式锁
5. 双重判定redis，因为多个请求都获取到了分布式锁 前面的请求加入redis了 后面的相同的请求就可以不用查库了
6. 查询路由表
7. 如果路由表中没有就插入第三步中的redis
8. 如果路由表有就根据路由表查长链接
9. 如果长链接已经失效就返回，否则返回长链接，并更新redis

```java
@SneakyThrows
@Override
public void restoreUrl(String shortUri, ServletRequest request, ServletResponse response) {
  
    String serverName = request.getServerName();
    String serverPort = Optional.of(request.getServerPort())
            .filter(each -> !Objects.equals(each, 80))
            .map(String::valueOf)
            .map(each -> ":" + each)
            .orElse("");
    String fullShortUrl = serverName + serverPort + "/" + shortUri;
    //首先查一次redis
    String originalLink = stringRedisTemplate.opsForValue().get(String.format(GOTO_SHORT_LINK_KEY, fullShortUrl));
    if (StrUtil.isNotBlank(originalLink)) {
        shortLinkStats(buildLinkStatsRecordAndSetUser(fullShortUrl, request, response));
        ((HttpServletResponse) response).sendRedirect(originalLink);
        return;
    }
    //查bloom过滤器
    boolean contains = shortUriCreateCachePenetrationBloomFilter.contains(fullShortUrl);
    if (!contains) {
        ((HttpServletResponse) response).sendRedirect("/page/notfound");
        return;
    }
    //查是否为空 在后面如果路由表中没有的话会加入redis 下一次来这里就会进行拦住
    String gotoIsNullShortLink = stringRedisTemplate.opsForValue().get(String.format(GOTO_IS_NULL_SHORT_LINK_KEY, fullShortUrl));
    if (StrUtil.isNotBlank(gotoIsNullShortLink)) {
        ((HttpServletResponse) response).sendRedirect("/page/notfound");
        return;
    }
    //加分布式锁
    RLock lock = redissonClient.getLock(String.format(LOCK_GOTO_SHORT_LINK_KEY, fullShortUrl));
    lock.lock();
    try {
        //双重判定 多个请求都获取到了分布式锁 前面的请求加入redis了 后面的相同的请求就可以不用查库了
        originalLink = stringRedisTemplate.opsForValue().get(String.format(GOTO_SHORT_LINK_KEY, fullShortUrl));
        if (StrUtil.isNotBlank(originalLink)) {
            shortLinkStats(buildLinkStatsRecordAndSetUser(fullShortUrl, request, response));
            ((HttpServletResponse) response).sendRedirect(originalLink);
            return;
        }
      	////因此空缓存也要做双重判定
        gotoIsNullShortLink = stringRedisTemplate.opsForValue().get(String.format(GOTO_IS_NULL_SHORT_LINK_KEY, fullShortUrl));
        if (StrUtil.isNotBlank(gotoIsNullShortLink)) {
            ((HttpServletResponse) response).sendRedirect("/page/notfound");
            return;
        }
        //查路由表有没有
        LambdaQueryWrapper<ShortLinkGotoDO> linkGotoQueryWrapper = Wrappers.lambdaQuery(ShortLinkGotoDO.class)
                .eq(ShortLinkGotoDO::getFullShortUrl, fullShortUrl);
        ShortLinkGotoDO shortLinkGotoDO = shortLinkGotoMapper.selectOne(linkGotoQueryWrapper);
        if (shortLinkGotoDO == null) {
            stringRedisTemplate.opsForValue().set(String.format(GOTO_IS_NULL_SHORT_LINK_KEY, fullShortUrl), "-", 30, TimeUnit.MINUTES);
            ((HttpServletResponse) response).sendRedirect("/page/notfound");
            return;
        }
        //路由表里面有 就可以根据路由表进行查询
        LambdaQueryWrapper<ShortLinkDO> queryWrapper = Wrappers.lambdaQuery(ShortLinkDO.class)
                .eq(ShortLinkDO::getGid, shortLinkGotoDO.getGid())
                .eq(ShortLinkDO::getFullShortUrl, fullShortUrl)
                .eq(ShortLinkDO::getDelFlag, 0)
                .eq(ShortLinkDO::getEnableStatus, 0);
        ShortLinkDO shortLinkDO = baseMapper.selectOne(queryWrapper);
        //有效期过期(shortLinkDO.getValidDate() != null && shortLinkDO.getValidDate().before(new Date())
        if (shortLinkDO == null || (shortLinkDO.getValidDate() != null && shortLinkDO.getValidDate().before(new Date()))) {
            stringRedisTemplate.opsForValue().set(String.format(GOTO_IS_NULL_SHORT_LINK_KEY, fullShortUrl), "-", 30, TimeUnit.MINUTES);
            ((HttpServletResponse) response).sendRedirect("/page/notfound");
            return;
        }
        stringRedisTemplate.opsForValue().set(
                String.format(GOTO_SHORT_LINK_KEY, fullShortUrl),
                shortLinkDO.getOriginUrl(),
                LinkUtil.getLinkCacheValidTime(shortLinkDO.getValidDate()), TimeUnit.MILLISECONDS
        );
        shortLinkStats(buildLinkStatsRecordAndSetUser(fullShortUrl, request, response));
        ((HttpServletResponse) response).sendRedirect(shortLinkDO.getOriginUrl());
    } finally {
        lock.unlock();
    }
}
```

**缓存预热**

在添加短链接的时候就加入redis中，设置一个有效期

#### 回收站管理

###### 加入回收站

逻辑删除，在数据库中设置状态为删除，在redis中需要删除缓存

```java
public void saveRecycleBin(RecycleBinSaveReqDTO requestParam) {
    LambdaUpdateWrapper<ShortLinkDO> updateWrapper = Wrappers.lambdaUpdate(ShortLinkDO.class)
            .eq(ShortLinkDO::getFullShortUrl, requestParam.getFullShortUrl())
            .eq(ShortLinkDO::getGid, requestParam.getGid())
            .eq(ShortLinkDO::getEnableStatus, 0)
            .eq(ShortLinkDO::getDelFlag, 0);
    ShortLinkDO shortLinkDO = ShortLinkDO.builder()
            .enableStatus(1)
            .build();
    baseMapper.update(shortLinkDO, updateWrapper);
    stringRedisTemplate.delete(String.format(GOTO_SHORT_LINK_KEY, requestParam.getFullShortUrl()));
}
```

###### 分页查询

这里的分页查询要考虑因为是在回收站，所以用户不必传入分组id，但由于在shortlink那张表里面用的是分组id进行的分表，如果直接搜的话就会全表查询，所以只能通过用户名找到他的所有的gid再进行查询

所以首先需要在admin项目中创建一个RecycleBinService层，RecycleBinController层调用RecycleBinService的pageRecycleBinShortLink方法获取分页查询，这个方法就会首先获取用户所有的gid，再将参数传给原来的分页查询方法进行查询

```java
    public Result<Page<ShortLinkPageRespDTO>> pageRecycleBinShortLink(ShortLinkRecycleBinPageReqDTO requestParam) {
        LambdaQueryWrapper<GroupDO> queryWrapper = Wrappers.lambdaQuery(GroupDO.class)
                .eq(GroupDO::getUsername, UserContext.getUsername())
                .eq(GroupDO::getDelFlag, 0);
        List<GroupDO> groupDOList = groupMapper.selectList(queryWrapper);
        if (CollUtil.isEmpty(groupDOList)) {
            throw new ServiceException("用户无分组信息");
        }
        requestParam.setGidList(groupDOList.stream().map(GroupDO::getGid).toList());
        return shortLinkActualRemoteService.pageRecycleBinShortLink(requestParam.getGidList(), requestParam.getCurrent(), requestParam.getSize());
    }
```

###### 恢复短链接

只需要把状态设置为1，并且将之前设置的redis的空白的key给删除

```java
public void recoverRecycleBin(RecycleBinRecoverReqDTO requestParam) {
    LambdaUpdateWrapper<ShortLinkDO> updateWrapper = Wrappers.lambdaUpdate(ShortLinkDO.class)
            .eq(ShortLinkDO::getFullShortUrl, requestParam.getFullShortUrl())
            .eq(ShortLinkDO::getGid, requestParam.getGid())
            .eq(ShortLinkDO::getEnableStatus, 1)
            .eq(ShortLinkDO::getDelFlag, 0);
    ShortLinkDO shortLinkDO = ShortLinkDO.builder()
            .enableStatus(0)
            .build();
    baseMapper.update(shortLinkDO, updateWrapper);
    stringRedisTemplate.delete(String.format(GOTO_IS_NULL_SHORT_LINK_KEY, requestParam.getFullShortUrl()));
}
```

###### 从回收站中移除

逻辑删除并且设置删除时间

```java
@Override
public void removeRecycleBin(RecycleBinRemoveReqDTO requestParam) {
    LambdaUpdateWrapper<ShortLinkDO> updateWrapper = Wrappers.lambdaUpdate(ShortLinkDO.class)
            .eq(ShortLinkDO::getFullShortUrl, requestParam.getFullShortUrl())
            .eq(ShortLinkDO::getGid, requestParam.getGid())
            .eq(ShortLinkDO::getEnableStatus, 1)
            .eq(ShortLinkDO::getDelTime, 0L)
            .eq(ShortLinkDO::getDelFlag, 0);
    ShortLinkDO delShortLinkDO = ShortLinkDO.builder()
            .delTime(System.currentTimeMillis())
            .build();
    delShortLinkDO.setDelFlag(1);
    baseMapper.update(delShortLinkDO, updateWrapper);
}
```

#### 短链接监控

###### pv统计

这里需要写sql语句，所以首先编写**LinkAccessStatsMapper**层

```java
public interface LinkAccessStatsMapper extends BaseMapper<LinkAccessStatsDO> {

    /**
     * 记录基础访问监控数据
     */
    @Insert("INSERT INTO t_link_access_stats (full_short_url, gid, date, pv, uv, uip, hour, weekday, create_time, update_time, del_flag) " +
            "VALUES( #{linkAccessStats.fullShortUrl}, #{linkAccessStats.gid}, #{linkAccessStats.date}, #{linkAccessStats.pv}, #{linkAccessStats.uv}, #{linkAccessStats.uip}, #{linkAccessStats.hour}, #{linkAccessStats.weekday}, NOW(), NOW(), 0) ON DUPLICATE KEY UPDATE pv = pv +  #{linkAccessStats.pv}, " +
            "uv = uv + #{linkAccessStats.uv}, " +
            " uip = uip + #{linkAccessStats.uip};")
    void shortLinkStats(@Param("linkAccessStats") LinkAccessStatsDO linkAccessStatsDO);
}
```

接着在service中编写插入函数，由于插入的地方可能没有gid，所以要进行判断，如果没有的话就需要首先去查库获取，接着就可以利用builder创建对象然后调用mapper

```java
    private void shortLinkStats(String fullShortUrl, String gid, ServletRequest request, ServletResponse response) {
        try {
            if (StrUtil.isBlank(gid)) {
                LambdaQueryWrapper<ShortLinkGotoDO> queryWrapper = Wrappers.lambdaQuery(ShortLinkGotoDO.class)
                        .eq(ShortLinkGotoDO::getFullShortUrl, fullShortUrl);
                ShortLinkGotoDO shortLinkGotoDO = shortLinkGotoMapper.selectOne(queryWrapper);
                gid = shortLinkGotoDO.getGid();
            }
            int hour = DateUtil.hour(new Date(), true);
            Week week = DateUtil.dayOfWeekEnum(new Date());
            int weekValue = week.getIso8601Value();
            LinkAccessStatsDO linkAccessStatsDO = LinkAccessStatsDO.builder()
                    .pv(1)
                    .uv(1)
                    .uip(1)
                    .hour(hour)
                    .weekday(weekValue)
                    .fullShortUrl(fullShortUrl)
                    .gid(gid)
                    .date(new Date())
                    .build();
            linkAccessStatsMapper.shortLinkStats(linkAccessStatsDO);
        } catch (Throwable ex) {
            log.error("短链接访问量统计异常", ex);
        }
    }
```

###### uv统计

统计用户的数量，一个用户可能多次访问，但是只算一次，所以这里需要对每个用户设置一个唯一表示，然后放到cookie中，每次访问的时候查cookie，看是否存在该cookie，并且从该cookie取出的值是否在redis中

```java
private void shortLinkStats(String fullShortUrl, String gid, ServletRequest request, ServletResponse response) {
        AtomicBoolean uvFirstFlag = new AtomicBoolean();
        Cookie[] cookies = ((HttpServletRequest) request).getCookies();
        try {
          	//这里定义一个runnable任务，用于如果没有cookie就添加cookie
            Runnable addResponseCookieTask = () -> {
                String uv = UUID.fastUUID().toString();
                Cookie uvCookie = new Cookie("uv", uv);
                uvCookie.setMaxAge(60 * 60 * 24 * 30);
              	//设置一个子路径的cookie，不然所有cookie都会堆到一个路径中
                uvCookie.setPath(StrUtil.sub(fullShortUrl, fullShortUrl.indexOf("/"), fullShortUrl.length()));
                ((HttpServletResponse) response).addCookie(uvCookie);
                uvFirstFlag.set(Boolean.TRUE);
                stringRedisTemplate.opsForSet().add("short-link:stats:uv:" + fullShortUrl, uv);
            };
            if (ArrayUtil.isNotEmpty(cookies)) {
              	//取出cookie判断是否是自己这个
                Arrays.stream(cookies)
                        .filter(each -> Objects.equals(each.getName(), "uv"))
                        .findFirst()
                        .map(Cookie::getValue)
                        .ifPresentOrElse(each -> {
                          	//利用redis的set结构 看是否能插入 来判断redis中是否有该cookie
                            Long added = stringRedisTemplate.opsForSet().add("short-link:stats:uv:" + fullShortUrl, each);
                            uvFirstFlag.set(added != null && added > 0L);
                        }, addResponseCookieTask);
            } else {
                addResponseCookieTask.run();
            }
            if (StrUtil.isBlank(gid)) {
                LambdaQueryWrapper<ShortLinkGotoDO> queryWrapper = Wrappers.lambdaQuery(ShortLinkGotoDO.class)
                        .eq(ShortLinkGotoDO::getFullShortUrl, fullShortUrl);
@@ -264,7 +293,7 @@ public class ShortLinkServiceImpl extends ServiceImpl<ShortLinkMapper, ShortLink
            int weekValue = week.getIso8601Value();
            LinkAccessStatsDO linkAccessStatsDO = LinkAccessStatsDO.builder()
                    .pv(1)
              			//根据flag来判断是否有该cookie
                    .uv(uvFirstFlag.get() ? 1 : 0)
                    .uip(1)
                    .hour(hour)
                    .weekday(weekValue)
              			.fullShortUrl(fullShortUrl)
                    .gid(gid)
                    .date(new Date())
                    .build();
            linkAccessStatsMapper.shortLinkStats(linkAccessStatsDO);
        } catch (Throwable ex) {
            log.error("短链接访问量统计异常", ex);
        }
    }
```

###### uip统计

这里需要获取每个请求的真实ip，然后拿去redis中比较即可

获取ip函数如下，可以在util类中创建

```java
    public static String getActualIp(HttpServletRequest request) {
        String ipAddress = request.getHeader("X-Forwarded-For");
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("HTTP_CLIENT_IP");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (ipAddress == null || ipAddress.isEmpty() || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getRemoteAddr();
        }
        return ipAddress;
    }
```

接着就可以shortLinkStats函数中对redis进行判断，然后看是否需要修改ip

###### 地区统计

**获取地区 API**

[IP-API.com - Geolocation API](https://ip-api.com/)

- 支持国内外IP地址
- 但是使用API限制较多

[IP定位-API文档-开发指南-Web服务 API | 高德地图API](https://lbs.amap.com/api/webservice/guide/api/ipconfig)

- 仅支持国内IP地址
- 使用API限制宽松

由于地区统计需要的参数比较多，所以先建了一个实体，并且建立对应的mapper层

```java
public interface LinkLocaleStatsMapper extends BaseMapper<LinkLocaleStatsDO> {

    /**
     * 记录地区访问监控数据
     */
    @Insert("INSERT INTO t_link_locale_stats (full_short_url, gid, date, cnt, country, province, city, adcode, create_time, update_time, del_flag) " +
            "VALUES( #{linkLocaleStats.fullShortUrl}, #{linkLocaleStats.gid}, #{linkLocaleStats.date}, #{linkLocaleStats.cnt}, #{linkLocaleStats.country}, #{linkLocaleStats.province}, #{linkLocaleStats.city}, #{linkLocaleStats.adcode}, NOW(), NOW(), 0) " +
            "ON DUPLICATE KEY UPDATE cnt = cnt +  #{linkLocaleStats.cnt};")
    void shortLinkLocaleState(@Param("linkLocaleStats") LinkLocaleStatsDO linkLocaleStatsDO);
}
```

接着在service层的shortLinkStats方法中设置

```java
Map<String, Object> localeParamMap = new HashMap<>();
						//statsLocaleAmapKey是网站获取，在配置文件中写，然后在该类用@value导入进来
            localeParamMap.put("key", statsLocaleAmapKey);
            localeParamMap.put("ip", remoteAddr);
						//AMAP_REMOTE_URL是定义在常量类中的url，也是api网站中获取
            String localeResultStr = HttpUtil.get(AMAP_REMOTE_URL, localeParamMap);
            JSONObject localeResultObj = JSON.parseObject(localeResultStr);
            String infoCode = localeResultObj.getString("infocode");
            if (StrUtil.isNotBlank(infoCode) && StrUtil.equals(infoCode, "10000")) {
                String province = localeResultObj.getString("province");
                boolean unknownFlag = StrUtil.equals(province, "[]");
                LinkLocaleStatsDO linkLocaleStatsDO = LinkLocaleStatsDO.builder()
                        .province(unknownFlag ? "未知" : province)
                        .city(unknownFlag ? "未知" : localeResultObj.getString("city"))
                        .adcode(unknownFlag ? "未知" : localeResultObj.getString("adcode"))
                        .cnt(1)
                        .fullShortUrl(fullShortUrl)
                        .country("中国")
                        .gid(gid)
                        .date(new Date())
                        .build();
                linkLocaleStatsMapper.shortLinkLocaleState(linkLocaleStatsDO);
            }
```

###### 操作系统统计

也是需要单独建表，接着定义mapper层

```java
public interface LinkOsStatsMapper extends BaseMapper<LinkOsStatsDO> {

    /**
     * 记录地区访问监控数据
     */
    @Insert("INSERT INTO t_link_os_stats (full_short_url, gid, date, cnt, os, create_time, update_time, del_flag) " +
            "VALUES( #{linkOsStats.fullShortUrl}, #{linkOsStats.gid}, #{linkOsStats.date}, #{linkOsStats.cnt}, #{linkOsStats.os}, NOW(), NOW(), 0) " +
            "ON DUPLICATE KEY UPDATE cnt = cnt +  #{linkOsStats.cnt};")
    void shortLinkOsState(@Param("linkOsStats") LinkOsStatsDO linkOsStatsDO);
}
```

接着和前面一个套路，在shortLinkStats方法中获取相关的参数，然后利用builder来构造出该对象，接着用mapper的方法插入即可

这是通过useragent获取os的方法，定义在util类中

```java
    public static String getOs(HttpServletRequest request) {
        String userAgent = request.getHeader("User-Agent");
        if (userAgent.toLowerCase().contains("windows")) {
            return "Windows";
        } else if (userAgent.toLowerCase().contains("mac")) {
            return "Mac OS";
        } else if (userAgent.toLowerCase().contains("linux")) {
            return "Linux";
        } else if (userAgent.toLowerCase().contains("android")) {
            return "Android";
        } else if (userAgent.toLowerCase().contains("iphone") || userAgent.toLowerCase().contains("ipad")) {
            return "iOS";
        } else {
            return "Unknown";
        }
    }
```

###### 短链接监控

前面是将各种参数记录在数据库中，现在就是要从数据库中拿出这些数据进行展示

```java
public ShortLinkStatsRespDTO oneShortLinkStats(ShortLinkStatsReqDTO requestParam) {
    checkGroupBelongToUser(requestParam.getGid());
    List<LinkAccessStatsDO> listStatsByShortLink = linkAccessStatsMapper.listStatsByShortLink(requestParam);
    if (CollUtil.isEmpty(listStatsByShortLink)) {
        return null;
    }
    // 基础访问数据
    LinkAccessStatsDO pvUvUidStatsByShortLink = linkAccessLogsMapper.findPvUvUidStatsByShortLink(requestParam);
    // 基础访问详情
    List<ShortLinkStatsAccessDailyRespDTO> daily = new ArrayList<>();
    List<String> rangeDates = DateUtil.rangeToList(DateUtil.parse(requestParam.getStartDate()), DateUtil.parse(requestParam.getEndDate()), DateField.DAY_OF_MONTH).stream()
            .map(DateUtil::formatDate)
            .toList();
    rangeDates.forEach(each -> listStatsByShortLink.stream()
            .filter(item -> Objects.equals(each, DateUtil.formatDate(item.getDate())))
            .findFirst()
            .ifPresentOrElse(item -> {
                ShortLinkStatsAccessDailyRespDTO accessDailyRespDTO = ShortLinkStatsAccessDailyRespDTO.builder()
                        .date(each)
                        .pv(item.getPv())
                        .uv(item.getUv())
                        .uip(item.getUip())
                        .build();
                daily.add(accessDailyRespDTO);
            }, () -> {
                ShortLinkStatsAccessDailyRespDTO accessDailyRespDTO = ShortLinkStatsAccessDailyRespDTO.builder()
                        .date(each)
                        .pv(0)
                        .uv(0)
                        .uip(0)
                        .build();
                daily.add(accessDailyRespDTO);
            }));
    // 地区访问详情（仅国内）
    List<ShortLinkStatsLocaleCNRespDTO> localeCnStats = new ArrayList<>();
    List<LinkLocaleStatsDO> listedLocaleByShortLink = linkLocaleStatsMapper.listLocaleByShortLink(requestParam);
    int localeCnSum = listedLocaleByShortLink.stream()
            .mapToInt(LinkLocaleStatsDO::getCnt)
            .sum();
    listedLocaleByShortLink.forEach(each -> {
        double ratio = (double) each.getCnt() / localeCnSum;
        double actualRatio = Math.round(ratio * 100.0) / 100.0;
        ShortLinkStatsLocaleCNRespDTO localeCNRespDTO = ShortLinkStatsLocaleCNRespDTO.builder()
                .cnt(each.getCnt())
                .locale(each.getProvince())
                .ratio(actualRatio)
                .build();
        localeCnStats.add(localeCNRespDTO);
    });
    // 小时访问详情
    List<Integer> hourStats = new ArrayList<>();
    List<LinkAccessStatsDO> listHourStatsByShortLink = linkAccessStatsMapper.listHourStatsByShortLink(requestParam);
    for (int i = 0; i < 24; i++) {
        AtomicInteger hour = new AtomicInteger(i);
        int hourCnt = listHourStatsByShortLink.stream()
                .filter(each -> Objects.equals(each.getHour(), hour.get()))
                .findFirst()
                .map(LinkAccessStatsDO::getPv)
                .orElse(0);
        hourStats.add(hourCnt);
    }
    // 高频访问IP详情
    List<ShortLinkStatsTopIpRespDTO> topIpStats = new ArrayList<>();
    List<HashMap<String, Object>> listTopIpByShortLink = linkAccessLogsMapper.listTopIpByShortLink(requestParam);
    listTopIpByShortLink.forEach(each -> {
        ShortLinkStatsTopIpRespDTO statsTopIpRespDTO = ShortLinkStatsTopIpRespDTO.builder()
                .ip(each.get("ip").toString())
                .cnt(Integer.parseInt(each.get("count").toString()))
                .build();
        topIpStats.add(statsTopIpRespDTO);
    });
    // 一周访问详情
    List<Integer> weekdayStats = new ArrayList<>();
    List<LinkAccessStatsDO> listWeekdayStatsByShortLink = linkAccessStatsMapper.listWeekdayStatsByShortLink(requestParam);
    for (int i = 1; i < 8; i++) {
        AtomicInteger weekday = new AtomicInteger(i);
        int weekdayCnt = listWeekdayStatsByShortLink.stream()
                .filter(each -> Objects.equals(each.getWeekday(), weekday.get()))
                .findFirst()
                .map(LinkAccessStatsDO::getPv)
                .orElse(0);
        weekdayStats.add(weekdayCnt);
    }
    // 浏览器访问详情
    List<ShortLinkStatsBrowserRespDTO> browserStats = new ArrayList<>();
    List<HashMap<String, Object>> listBrowserStatsByShortLink = linkBrowserStatsMapper.listBrowserStatsByShortLink(requestParam);
    int browserSum = listBrowserStatsByShortLink.stream()
            .mapToInt(each -> Integer.parseInt(each.get("count").toString()))
            .sum();
    listBrowserStatsByShortLink.forEach(each -> {
        double ratio = (double) Integer.parseInt(each.get("count").toString()) / browserSum;
        double actualRatio = Math.round(ratio * 100.0) / 100.0;
        ShortLinkStatsBrowserRespDTO browserRespDTO = ShortLinkStatsBrowserRespDTO.builder()
                .cnt(Integer.parseInt(each.get("count").toString()))
                .browser(each.get("browser").toString())
                .ratio(actualRatio)
                .build();
        browserStats.add(browserRespDTO);
    });
    // 访客访问类型详情
    List<ShortLinkStatsUvRespDTO> uvTypeStats = new ArrayList<>();
    HashMap<String, Object> findUvTypeByShortLink = linkAccessLogsMapper.findUvTypeCntByShortLink(requestParam);
    int oldUserCnt = Integer.parseInt(
            Optional.ofNullable(findUvTypeByShortLink)
                    .map(each -> each.get("oldUserCnt"))
                    .map(Object::toString)
                    .orElse("0")
    );
    int newUserCnt = Integer.parseInt(
            Optional.ofNullable(findUvTypeByShortLink)
                    .map(each -> each.get("newUserCnt"))
                    .map(Object::toString)
                    .orElse("0")
    );
    int uvSum = oldUserCnt + newUserCnt;
    double oldRatio = (double) oldUserCnt / uvSum;
    double actualOldRatio = Math.round(oldRatio * 100.0) / 100.0;
    double newRatio = (double) newUserCnt / uvSum;
    double actualNewRatio = Math.round(newRatio * 100.0) / 100.0;
    ShortLinkStatsUvRespDTO newUvRespDTO = ShortLinkStatsUvRespDTO.builder()
            .uvType("newUser")
            .cnt(newUserCnt)
            .ratio(actualNewRatio)
            .build();
    uvTypeStats.add(newUvRespDTO);
    ShortLinkStatsUvRespDTO oldUvRespDTO = ShortLinkStatsUvRespDTO.builder()
            .uvType("oldUser")
            .cnt(oldUserCnt)
            .ratio(actualOldRatio)
            .build();
    uvTypeStats.add(oldUvRespDTO);
    // 访问设备类型详情
    List<ShortLinkStatsDeviceRespDTO> deviceStats = new ArrayList<>();
    List<LinkDeviceStatsDO> listDeviceStatsByShortLink = linkDeviceStatsMapper.listDeviceStatsByShortLink(requestParam);
    int deviceSum = listDeviceStatsByShortLink.stream()
            .mapToInt(LinkDeviceStatsDO::getCnt)
            .sum();
    listDeviceStatsByShortLink.forEach(each -> {
        double ratio = (double) each.getCnt() / deviceSum;
        double actualRatio = Math.round(ratio * 100.0) / 100.0;
        ShortLinkStatsDeviceRespDTO deviceRespDTO = ShortLinkStatsDeviceRespDTO.builder()
                .cnt(each.getCnt())
                .device(each.getDevice())
                .ratio(actualRatio)
                .build();
        deviceStats.add(deviceRespDTO);
    });
    // 访问网络类型详情
    List<ShortLinkStatsNetworkRespDTO> networkStats = new ArrayList<>();
    List<LinkNetworkStatsDO> listNetworkStatsByShortLink = linkNetworkStatsMapper.listNetworkStatsByShortLink(requestParam);
    int networkSum = listNetworkStatsByShortLink.stream()
            .mapToInt(LinkNetworkStatsDO::getCnt)
            .sum();
    listNetworkStatsByShortLink.forEach(each -> {
        double ratio = (double) each.getCnt() / networkSum;
        double actualRatio = Math.round(ratio * 100.0) / 100.0;
        ShortLinkStatsNetworkRespDTO networkRespDTO = ShortLinkStatsNetworkRespDTO.builder()
                .cnt(each.getCnt())
                .network(each.getNetwork())
                .ratio(actualRatio)
                .build();
        networkStats.add(networkRespDTO);
    });
    return ShortLinkStatsRespDTO.builder()
            .pv(pvUvUidStatsByShortLink.getPv())
            .uv(pvUvUidStatsByShortLink.getUv())
            .uip(pvUvUidStatsByShortLink.getUip())
            .daily(daily)
            .localeCnStats(localeCnStats)
            .hourStats(hourStats)
            .topIpStats(topIpStats)
            .weekdayStats(weekdayStats)
            .browserStats(browserStats)
            .osStats(osStats)
            .uvTypeStats(uvTypeStats)
            .deviceStats(deviceStats)
            .networkStats(networkStats)
            .build();
}
```

###### 分页访问记录

这里比较麻烦的是写sql语句

首先写service层代码

```java
public IPage<ShortLinkStatsAccessRecordRespDTO> shortLinkStatsAccessRecord(ShortLinkStatsAccessRecordReqDTO requestParam) {
        LambdaQueryWrapper<LinkAccessLogsDO> queryWrapper = Wrappers.lambdaQuery(LinkAccessLogsDO.class)
                .eq(LinkAccessLogsDO::getGid, requestParam.getGid())
                .eq(LinkAccessLogsDO::getFullShortUrl, requestParam.getFullShortUrl())
            	//这里要用between
                .between(LinkAccessLogsDO::getCreateTime, requestParam.getStartDate(), requestParam.getEndDate())
                .eq(LinkAccessLogsDO::getDelFlag, 0)
                .orderByDesc(LinkAccessLogsDO::getCreateTime);
        IPage<LinkAccessLogsDO> linkAccessLogsDOIPage = linkAccessLogsMapper.selectPage(requestParam, queryWrapper);
        IPage<ShortLinkStatsAccessRecordRespDTO> actualResult = linkAccessLogsDOIPage.convert(each -> BeanUtil.toBean(each, ShortLinkStatsAccessRecordRespDTO.class));
        List<String> userAccessLogsList = actualResult.getRecords().stream()
                .map(ShortLinkStatsAccessRecordRespDTO::getUser)
                .toList();
        List<Map<String, Object>> uvTypeList = linkAccessLogsMapper.selectUvTypeByUsers(
                requestParam.getGid(),
                requestParam.getFullShortUrl(),
                requestParam.getStartDate(),
                requestParam.getEndDate(),
                userAccessLogsList
        );
        actualResult.getRecords().forEach(each -> {
            String uvType = uvTypeList.stream()
                    .filter(item -> Objects.equals(each.getUser(), item.get("user")))
                    .findFirst()
                    .map(item -> item.get("UvType"))
                    .map(Object::toString)
                    .orElse("旧访客");
            each.setUvType(uvType);
        });
        return actualResult;
    }
```

接着调用mapper层手写的sql语句

这里要注意，由于使用了动态语句，所以要加script，分页才能正确解析

```java
    @Select("<script> " +
            "SELECT " +
            "    user, " +
            "    CASE " +
            "        WHEN MIN(create_time) BETWEEN #{startDate} AND #{endDate} THEN '新访客' " +
            "        ELSE '老访客' " +
            "    END AS uvType " +
            "FROM " +
            "    t_link_access_logs " +
            "WHERE " +
            "    full_short_url = #{fullShortUrl} " +
            "    AND gid = #{gid} " +
            "    AND user IN " +
            "    <foreach item='item' index='index' collection='userAccessLogsList' open='(' separator=',' close=')'> " +
            "        #{item} " +
            "    </foreach> " +
            "GROUP BY " +
            "    user;" +
            "    </script>"
    )
    List<Map<String, Object>> selectUvTypeByUsers(
            @Param("gid") String gid,
            @Param("fullShortUrl") String fullShortUrl,
            @Param("startDate") String startDate,
            @Param("endDate") String endDate,
            @Param("userAccessLogsList") List<String> userAccessLogsList
    );
```

###### 削峰短链接监控功能

之前是在短链接进行跳转的时候加入了短链接监控的功能，会同步的在数据库中对相关的pv、uv等参数进行修改，考虑到如果是海量的数据访问短链接，直接访问数据库，会导致数据库负载变高，甚至宕机，为此需要引入消息队列削峰

![image-20240620225131722](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240620225131722.png)

使用redis stream作为消息队列

首先创建redis stream消息队列配置类

```java
@Configuration
@RequiredArgsConstructor
public class RedisStreamConfiguration {

    private final RedisConnectionFactory redisConnectionFactory;
    private final ShortLinkStatsSaveConsumer shortLinkStatsSaveConsumer;

    @Value("${spring.data.redis.channel-topic.short-link-stats}")
    private String topic;
    @Value("${spring.data.redis.channel-topic.short-link-stats-group}")
    private String group;

    @Bean
    public ExecutorService asyncStreamConsumer() {
        AtomicInteger index = new AtomicInteger();
        int processors = Runtime.getRuntime().availableProcessors();
        return new ThreadPoolExecutor(processors,
                processors + processors >> 1,
                60,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(),
                runnable -> {
                    Thread thread = new Thread(runnable);
                    thread.setName("stream_consumer_short-link_stats_" + index.incrementAndGet());
                    thread.setDaemon(true);
                    return thread;
                }
        );
    }

    @Bean(initMethod = "start", destroyMethod = "stop")
    public StreamMessageListenerContainer<String, MapRecord<String, String, String>> streamMessageListenerContainer(ExecutorService asyncStreamConsumer) {
        StreamMessageListenerContainer.StreamMessageListenerContainerOptions<String, MapRecord<String, String, String>> options =
                StreamMessageListenerContainer.StreamMessageListenerContainerOptions
                        .builder()
                        // 一次最多获取多少条消息
                        .batchSize(10)
                        // 执行从 Stream 拉取到消息的任务流程
                        .executor(asyncStreamConsumer)
                        // 如果没有拉取到消息，需要阻塞的时间。不能大于 ${spring.data.redis.timeout}，否则会超时
                        .pollTimeout(Duration.ofSeconds(3))
                        .build();
        StreamMessageListenerContainer<String, MapRecord<String, String, String>> streamMessageListenerContainer =
                StreamMessageListenerContainer.create(redisConnectionFactory, options);
        streamMessageListenerContainer.receiveAutoAck(Consumer.from(group, "stats-consumer"),
                StreamOffset.create(topic, ReadOffset.lastConsumed()), shortLinkStatsSaveConsumer);
        return streamMessageListenerContainer;
    }
}
```

接着创建短链接监控状态保存消息队列消费者

```java
public class ShortLinkStatsSaveConsumer implements StreamListener<String, MapRecord<String, String, String>> {

    private final ShortLinkMapper shortLinkMapper;
    private final ShortLinkGotoMapper shortLinkGotoMapper;
    private final RedissonClient redissonClient;
    private final LinkAccessStatsMapper linkAccessStatsMapper;
    private final LinkLocaleStatsMapper linkLocaleStatsMapper;
    private final LinkOsStatsMapper linkOsStatsMapper;
    private final LinkBrowserStatsMapper linkBrowserStatsMapper;
    private final LinkAccessLogsMapper linkAccessLogsMapper;
    private final LinkDeviceStatsMapper linkDeviceStatsMapper;
    private final LinkNetworkStatsMapper linkNetworkStatsMapper;
    private final LinkStatsTodayMapper linkStatsTodayMapper;
    private final DelayShortLinkStatsProducer delayShortLinkStatsProducer;
    private final StringRedisTemplate stringRedisTemplate;

    @Value("${short-link.stats.locale.amap-key}")
    private String statsLocaleAmapKey;

    @Override
    public void onMessage(MapRecord<String, String, String> message) {
        String stream = message.getStream();
        RecordId id = message.getId();
        Map<String, String> producerMap = message.getValue();
        String fullShortUrl = producerMap.get("fullShortUrl");
        if (StrUtil.isNotBlank(fullShortUrl)) {
            String gid = producerMap.get("gid");
            ShortLinkStatsRecordDTO statsRecord = JSON.parseObject(producerMap.get("statsRecord"), ShortLinkStatsRecordDTO.class);
            actualSaveShortLinkStats(fullShortUrl, gid, statsRecord);
        }
        stringRedisTemplate.opsForStream().delete(Objects.requireNonNull(stream), id.getValue());
    }
		//这个方法就是之前的监控插入数据库的方法，将逻辑抽到这里面来了
    public void actualSaveShortLinkStats(String fullShortUrl, String gid, ShortLinkStatsRecordDTO statsRecord) {
        fullShortUrl = Optional.ofNullable(fullShortUrl).orElse(statsRecord.getFullShortUrl());
        RReadWriteLock readWriteLock = redissonClient.getReadWriteLock(String.format(LOCK_GID_UPDATE_KEY, fullShortUrl));
        RLock rLock = readWriteLock.readLock();
        if (!rLock.tryLock()) {
            delayShortLinkStatsProducer.send(statsRecord);
            return;
        }
        try {
            if (StrUtil.isBlank(gid)) {
                LambdaQueryWrapper<ShortLinkGotoDO> queryWrapper = Wrappers.lambdaQuery(ShortLinkGotoDO.class)
                        .eq(ShortLinkGotoDO::getFullShortUrl, fullShortUrl);
                ShortLinkGotoDO shortLinkGotoDO = shortLinkGotoMapper.selectOne(queryWrapper);
                gid = shortLinkGotoDO.getGid();
            }
            int hour = DateUtil.hour(new Date(), true);
            Week week = DateUtil.dayOfWeekEnum(new Date());
            int weekValue = week.getIso8601Value();
						。。。。。
```

创建短链接监控状态保存消息队列生产者

```java
@Component
@RequiredArgsConstructor
public class ShortLinkStatsSaveProducer {

    private final StringRedisTemplate stringRedisTemplate;

    @Value("${spring.data.redis.channel-topic.short-link-stats}")
    private String topic;

    /**
     * 发送延迟消费短链接统计
     */
    public void send(Map<String, String> producerMap) {
        stringRedisTemplate.opsForStream().add(topic, producerMap);
    }
}
```

不过这里存在重复消费问题

当消息队列出现重复消费问题情况下，应该如何保障数据的准确性？

- 网络问题
- 生产重试

**解决方式**

幂等

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240621152312663-20240621152333290.png)

具体思路，在消费者方法中每次调用都会设置一个带id一个redis的key记录，有限时间为10min，用这个来判断消息是否被消费过

首先创建一个handler，用户设置redis

```java
public class MessageQueueIdempotentHandler {

    private final StringRedisTemplate stringRedisTemplate;

    private static final String IDEMPOTENT_KEY_PREFIX = "short-link:idempotent:";

    /**
     * 判断当前消息是否消费过
     *
     * @param messageId 消息唯一标识
     * @return 消息是否消费过
     */
    public boolean isMessageProcessed(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        return Boolean.TRUE.equals(stringRedisTemplate.opsForValue().setIfAbsent(key, "0", 2, TimeUnit.MINUTES));
    }

    /**
     * 判断消息消费流程是否执行完成
     *
     * @param messageId 消息唯一标识
     * @return 消息是否执行完成
     */
    public boolean isAccomplish(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        return Objects.equals(stringRedisTemplate.opsForValue().get(key), "1");
    }

    /**
     * 设置消息流程执行完成
     *
     * @param messageId 消息唯一标识
     */
    public void setAccomplish(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        stringRedisTemplate.opsForValue().set(key, "1", 2, TimeUnit.MINUTES);
    }

    /**
     * 如果消息处理遇到异常情况，删除幂等标识
     *
     * @param messageId 消息唯一标识
     */
    public void delMessageProcessed(String messageId) {
        String key = IDEMPOTENT_KEY_PREFIX + messageId;
        stringRedisTemplate.delete(key);
    }
}
```

接着在消息队列消费者类的onMessage中进行判断

```java
public void onMessage(MapRecord<String, String, String> message) {
        String stream = message.getStream();
        RecordId id = message.getId();
        if (!messageQueueIdempotentHandler.isMessageProcessed(id.toString())) {
            // 判断当前的这个消息流程是否执行完成
            if (messageQueueIdempotentHandler.isAccomplish(id.toString())) {
                return;
            }
            throw new ServiceException("消息未完成流程，需要消息队列重试");
        }
        try {
            Map<String, String> producerMap = message.getValue();
            String fullShortUrl = producerMap.get("fullShortUrl");
            if (StrUtil.isNotBlank(fullShortUrl)) {
                String gid = producerMap.get("gid");
                ShortLinkStatsRecordDTO statsRecord = JSON.parseObject(producerMap.get("statsRecord"), ShortLinkStatsRecordDTO.class);
                actualSaveShortLinkStats(fullShortUrl, gid, statsRecord);
            }
            stringRedisTemplate.opsForStream().delete(Objects.requireNonNull(stream), id.getValue());
        } catch (Throwable ex) {
            // 某某某情况宕机了
            messageQueueIdempotentHandler.delMessageProcessed(id.toString());
            log.error("记录短链接监控消费异常", ex);
          	//这里要加一个抛出，否则会走最后set完成这个逻辑，就会让这个消息无法被消费
          	throw ex;
        }
        messageQueueIdempotentHandler.setAccomplish(id.toString());
    }
```

但也有可能走不到catch这一步，在try中就中断了，这样因为之前加了redis，所以消息队列就会在前面判断的时候返回，就无法插入数据库中

解决思路是：在原来redis中设置值，如果等于0表示还未消费，等于1表示已消费，在try结束位置来设置为1，在最前面的if判断中再加入一个对值的判断，就可以解决这个问题

这样当在try中发生突然中断，redis的值就没有设置成功，这样即便进入if，也会一直抛出异常，直到redis过期，这个时候就不会再进入最外面的if，直接进入try进行消费

```java
if (statsRecord != null) {
                                if (!messageQueueIdempotentHandler.isMessageProcessed(statsRecord.getKeys())) {
                                    // 判断当前的这个消息流程是否执行完成
                                    if (messageQueueIdempotentHandler.isAccomplish(statsRecord.getKeys())) {
                                        return;
                                    }
                                    throw new ServiceException("消息未完成流程，需要消息队列重试");
                                }
                                try {
                                    shortLinkService.shortLinkStats(null, null, statsRecord);
                                } catch (Throwable ex) {
                                    messageQueueIdempotentHandler.delMessageProcessed(statsRecord.getKeys());
                                    log.error("延迟记录短链接监控消费异常", ex);
                                }
                                messageQueueIdempotentHandler.setAccomplish(statsRecord.getKeys());
                                continue;
                            }
```



#### 功能扩展

###### 添加白名单

考虑到原始链接需要是合法的链接，如果是对链接进行判断是否合法就太麻烦了，所以就采用添加白名单的方式

1. 在application配置文件中写入允许链接的域名

2. 然后在创建短链接和修改短链接的时候就要对域名进行验证

   ```java
   public static String extractDomain(String url) {
           String domain = null;
           try {
               URI uri = new URI(url);
               String host = uri.getHost();
               if (StrUtil.isNotBlank(host)) {
                   domain = host;
                   if (domain.startsWith("www.")) {
                       domain = host.substring(4);
                   }
               }
           } catch (Exception ignored) {
           }
           return domain;
       }
       
   private void verificationWhitelist(String originUrl) {
           Boolean enable = gotoDomainWhiteListConfiguration.getEnable();
           if (enable == null || !enable) {
               return;
           }
           String domain = LinkUtil.extractDomain(originUrl);
           if (StrUtil.isBlank(domain)) {
               throw new ClientException("跳转链接填写错误");
           }
           List<String> details = gotoDomainWhiteListConfiguration.getDetails();
           if (!details.contains(domain)) {
               throw new ClientException("演示环境为避免恶意攻击，请生成以下网站跳转链接：" + gotoDomainWhiteListConfiguration.getNames());
           }
       }
   ```

###### 创建分组限制

默认创建最多20个分组，因为后面部署是微服务，创建分组的时候需要进行判断当前分组数量是否超过限制，但是syn是锁不了的，所以要使用分布式锁

```java
public void saveGroup(String username, String groupName) {
        RLock lock = redissonClient.getLock(String.format(LOCK_GROUP_CREATE_KEY, username));
        lock.lock();
        try {
            LambdaQueryWrapper<GroupDO> queryWrapper = Wrappers.lambdaQuery(GroupDO.class)
                    .eq(GroupDO::getUsername, username)
                    .eq(GroupDO::getDelFlag, 0);
            List<GroupDO> groupDOList = baseMapper.selectList(queryWrapper);
            if (CollUtil.isNotEmpty(groupDOList) && groupDOList.size() == groupMaxNum) {
                throw new ClientException(String.format("已超出最大分组数：%d", groupMaxNum));
            }
            String gid;
            do {
                gid = RandomGenerator.generateRandom();
            } while (!hasGid(username, gid));
            GroupDO groupDO = GroupDO.builder()
                    .gid(gid)
                    .sortOrder(0)
                    .username(username)
                    .name(groupName)
                    .build();
            baseMapper.insert(groupDO);
        } finally {
            lock.unlock();
        }
    }
```

###### 流量风控

**短链接后管**

根据登录用户做出控制，比如 x 秒请求后管系统的频率最多 x 次。

实现原理也比较简单，**通过 Redis `increment` 命令对一个数据进行递增**，如果超过 x 次就会返回失败。这里有个细节就是这个周期是 x 秒，需要对 Redis 的 Key 设置 x 秒有效期。

但是 Redis 中对于 `increment` 命令是没有提供过期命令的，这就需要两步操作，进而出现原子性问题

为此，我们需要通过 LUA 脚本来保证原子性

```lua
-- 设置用户访问频率限制的参数
local username = KEYS[1]
local timeWindow = tonumber(ARGV[1]) -- 时间窗口，单位：秒

-- 构造 Redis 中存储用户访问次数的键名
local accessKey = "short-link:user-flow-risk-control:" .. username

-- 原子递增访问次数，并获取递增后的值
local currentAccessCount = redis.call("INCR", accessKey)

-- 设置键的过期时间
redis.call("EXPIRE", accessKey, timeWindow)

-- 返回当前访问次数
return currentAccessCount
```

这里写的lua脚本是存在一些问题的，考虑下面场景：

这个lua脚本的逻辑是设置时间窗口内最多能访问多少次，如果超出限制就报错，但如果在即将过期时访问了，刷新了过期时间，就会一直递增

```java
-- 设置用户访问频率限制的参数
local username = KEYS[1]
local timeWindow = tonumber(ARGV[1]) -- 时间窗口，单位：秒

-- 构造 Redis 中存储用户访问次数的键名
local accessKey = "short-link:user-flow-risk-control:" .. username

-- 原子递增访问次数，并获取递增后的值
local currentAccessCount = redis.call("INCR", accessKey)

-- 设置键的过期时间
if currentAccessCount == 1 then
    redis.call("EXPIRE", accessKey, timeWindow)
end

-- 返回当前访问次数
return currentAccessCount
```

接着设置一个过滤器，来执行lua脚本

```java
public class UserFlowRiskControlFilter implements Filter {

    private final StringRedisTemplate stringRedisTemplate;
    private final UserFlowRiskControlConfiguration userFlowRiskControlConfiguration;

    private static final String USER_FLOW_RISK_CONTROL_LUA_SCRIPT_PATH = "lua/user_flow_risk_control.lua";

    @SneakyThrows
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource(USER_FLOW_RISK_CONTROL_LUA_SCRIPT_PATH)));
        redisScript.setResultType(Long.class);
        String username = Optional.ofNullable(UserContext.getUsername()).orElse("other");
        Long result = null;
        try {
            result = stringRedisTemplate.execute(redisScript, Lists.newArrayList(username), userFlowRiskControlConfiguration.getTimeWindow());
        } catch (Throwable ex) {
            log.error("执行用户请求流量限制LUA脚本出错", ex);
            returnJson((HttpServletResponse) response, JSON.toJSONString(Results.failure(new ClientException(FLOW_LIMIT_ERROR))));
        }
        if (result == null || result > userFlowRiskControlConfiguration.getMaxAccessCount()) {
            returnJson((HttpServletResponse) response, JSON.toJSONString(Results.failure(new ClientException(FLOW_LIMIT_ERROR))));
        }
        filterChain.doFilter(request, response);
    }

    private void returnJson(HttpServletResponse response, String json) throws Exception {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html; charset=utf-8");
        try (PrintWriter writer = response.getWriter()) {
            writer.print(json);
        }
    }
}
```

接着在userconfig里面配置该过滤器，并设置其与其他过滤器之间的顺序

```java
    @Bean
    @ConditionalOnProperty(name = "short-link.flow-limit.enable", havingValue = "true")
    public FilterRegistrationBean<UserFlowRiskControlFilter> globalUserFlowRiskControlFilter(
            StringRedisTemplate stringRedisTemplate,
            UserFlowRiskControlConfiguration userFlowRiskControlConfiguration) {
        FilterRegistrationBean<UserFlowRiskControlFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new UserFlowRiskControlFilter(stringRedisTemplate, userFlowRiskControlConfiguration));
        registration.addUrlPatterns("/*");
        registration.setOrder(10);
        return registration;
    }
```

还有一种方式可以实现流量风控

**Sentinel**

Sentinel 是面向分布式、多语言异构化服务架构的流量治理组件，主要以流量为切入点，从流量控制、流量路由、熔断降级、系统自适应保护等多个维度来帮助用户保障微服务的稳定性

单机模式下使用sentinel步骤

1. 导入依赖

   ```java
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   
   <dependency>
       <groupId>com.alibaba.csp</groupId>
       <artifactId>sentinel-annotation-aspectj</artifactId>
   </dependency>
   ```

2. 定义接口规则

   创建一个配置类定义接口规则

   ```java
   @Component
   public class SentinelRuleConfig implements InitializingBean {
   
       @Override
       public void afterPropertiesSet() throws Exception {
           List<FlowRule> rules = new ArrayList<>();
           FlowRule createOrderRule = new FlowRule();
           createOrderRule.setResource("create_short-link");
           createOrderRule.setGrade(RuleConstant.FLOW_GRADE_QPS);
           createOrderRule.setCount(1);
           rules.add(createOrderRule);
           FlowRuleManager.loadRules(rules);
       }
   }
   ```

   定义handler，即如果触发风控，设置降级策略

   ```java
   public class CustomBlockHandler {
   
       public static Result<ShortLinkCreateRespDTO> createShortLinkBlockHandlerMethod(ShortLinkCreateReqDTO requestParam, BlockException exception) {
           return new Result<ShortLinkCreateRespDTO>().setCode("B100000").setMessage("当前访问网站人数过多，请稍后再试...");
       }
   }
   ```

3. 在代码中引入Sentinel注解控流规则

在微服务中引入Sentinel

[官方文档](https://sentinelguard.io/zh-cn/docs/dashboard.html)

启动Sentinel 控制台，删除 Sentinel 定义的相关规则代码，加入以下配置即可

删除的规则配置，在 Sentinel 中进行配置

```java
spring:
    sentinel:
      transport:
        dashboard: localhost:8686
        port: 8719
```

#### 微服务改造

分为以下四个步骤

1. 下载Nacos

2. 服务中引入Nacos进行服务注册

   服务自主发起注册

   ```java
   <dependency>
     <groupId>com.alibaba.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

   如果是调用方，需要引入 OpenFeign 组件

   ```java
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   
   <!-- openfeign 已不再提供默认负载均衡器 -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-loadbalancer</artifactId>
   </dependency>
   ```

   启动类添加 Nacos 注册中心注解

   ```java
   @EnableDiscoveryClient
   ```

   配置yaml

   ```java
   spring:
     application:
       name: short-link-admin
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
   ```

3. 改造现有代码，通过OpenFeign调用

   在admin项目中创建一个调用project项目的接口

   在接口中编写调用project项目中的方法

   ```java
   @FeignClient("short-link-project")
   public interface ShortLinkActualRemoteService {
   	// 调用接口
     @PostMapping("/api/short-link/v1/create")
       Result<ShortLinkCreateRespDTO> createShortLink(@RequestBody ShortLinkCreateReqDTO requestParam);
   
   }
   ```

   在admin业务中调用这个接口中的方法，就可以实现Feign调用

   ```java
   private final ShortLinkActualRemoteService shortLinkActualRemoteService;
   
   shortLinkActualRemoteService.xxx();
   ```

4. 重构网关 spring cloud gateway 

   网关的作用

   - 路由管理&服务发现困难。
   - 安全性难以管理：https 访问、黑白名单、用户登录和数据请求加密放篡改等。
   - 负载均衡问题。
   - 监控和日志难以集中管理。
   - 缺乏统一的 API 管理

   ![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240622215742075-20240622215753942.png)

   在**gateway项目**中进行操作

   首先引入依赖

   ```java
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-gateway</artifactId>
       </dependency>
   
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-loadbalancer</artifactId>
       </dependency>
   
       <!-- Unable to load io.netty.resolver.dns.macos.MacOSDnsServerAddressStreamProvider xxx -->
       <dependency>
           <groupId>io.netty</groupId>
           <artifactId>netty-all</artifactId>
       </dependency>
   
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
   
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-data-redis</artifactId>
       </dependency>
   
       <dependency>
           <groupId>com.alibaba.fastjson2</groupId>
           <artifactId>fastjson2</artifactId>
       </dependency>
   </dependencies>
   
   <build>
       <finalName>${project.artifactId}</finalName>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
               <executions>
                   <execution>
                       <goals>
                           <goal>repackage</goal>
                       </goals>
                   </execution>
               </executions>
           </plugin>
       </plugins>
   </build>
   ```

   创建启动类，添加网关配置文件

   ```java
   server:
     port: 8000
   spring:
     application:
       name: short-link-gateway
     data:
       redis:
         host: 127.0.0.1
         port: 6379
         password: 123456
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
       gateway:
         routes:
   				//配置两个项目的路由
           - id: short-link-admin
             uri: lb://short-link-admin/api/short-link/admin/**
             predicates:
               - Path=/api/short-link/admin/**
             filters:
               - name: TokenValidate
                 args:
                   whitePathList:
                     - /api/short-link/admin/v1/user/login
                     - /api/short-link/admin/v1/user/has-username
   
           - id: short-link-project
             uri: lb://short-link-project/api/short-link/**
             predicates:
               - Path=/api/short-link/**
             filters:
               - name: TokenValidate
   ```

   设置用户登录拦截器

   ```java
   /**
    * SpringCloud Gateway Token 拦截器
    */
   @Component
   public class TokenValidateGatewayFilterFactory extends AbstractGatewayFilterFactory<Config> {
   
       private final StringRedisTemplate stringRedisTemplate;
   
       public TokenValidateGatewayFilterFactory(StringRedisTemplate stringRedisTemplate) {
           super(Config.class);
           this.stringRedisTemplate = stringRedisTemplate;
       }
   
       @Override
       public GatewayFilter apply(Config config) {
           return (exchange, chain) -> {
               ServerHttpRequest request = exchange.getRequest();
               String requestPath = request.getPath().toString();
               String requestMethod = request.getMethod().name();
               if (!isPathInWhiteList(requestPath, requestMethod, config.getWhitePathList())) {
                   String username = request.getHeaders().getFirst("username");
                   String token = request.getHeaders().getFirst("token");
                   Object userInfo;
                   if (StringUtils.hasText(username) && StringUtils.hasText(token) && (userInfo = stringRedisTemplate.opsForHash().get("short-link:login:" + username, token)) != null) {
                       JSONObject userInfoJsonObject = JSON.parseObject(userInfo.toString());
                       ServerHttpRequest.Builder builder = exchange.getRequest().mutate().headers(httpHeaders -> {
                           httpHeaders.set("userId", userInfoJsonObject.getString("id"));
                           httpHeaders.set("realName", URLEncoder.encode(userInfoJsonObject.getString("realName"), StandardCharsets.UTF_8));
                       });
                       return chain.filter(exchange.mutate().request(builder.build()).build());
                   }
                   ServerHttpResponse response = exchange.getResponse();
                   response.setStatusCode(HttpStatus.UNAUTHORIZED);
                   return response.writeWith(Mono.fromSupplier(() -> {
                       DataBufferFactory bufferFactory = response.bufferFactory();
                       GatewayErrorResult resultMessage = GatewayErrorResult.builder()
                               .status(HttpStatus.UNAUTHORIZED.value())
                               .message("Token validation error")
                               .build();
                       return bufferFactory.wrap(JSON.toJSONString(resultMessage).getBytes());
                   }));
               }
               return chain.filter(exchange);
           };
       }
   
       private boolean isPathInWhiteList(String requestPath, String requestMethod, List<String> whitePathList) {
           return (!CollectionUtils.isEmpty(whitePathList) && whitePathList.stream().anyMatch(requestPath::startsWith)) || (Objects.equals(requestPath, "/api/short-link/admin/v1/user") && Objects.equals(requestMethod, "POST"));
       }
   }
   ```

   这里面的config是自己创建的过滤器配置类

   ```java
   /**
    * 过滤器配置
    */
   @Data
   public class Config {
   
       /**
        * 白名单前置路径
        */
       private List<String> whitePathList;
   }
   
   ```

   因为现在是用网关作为一个用户登录的拦截，所以之前的amdin项目里面的过滤器就不需要再验证一遍了，就可以只需要将用户信息加入usercontext即可

   ```java
   @RequiredArgsConstructor
   public class UserTransmitFilter implements Filter {
   
       @SneakyThrows
       @Override
       public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) {
           HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
           String username = httpServletRequest.getHeader("username");
           if (StrUtil.isNotBlank(username)) {
               String userId = httpServletRequest.getHeader("userId");
               String realName = httpServletRequest.getHeader("realName");
               UserInfoDTO userInfoDTO = new UserInfoDTO(userId, username, realName);
               UserContext.setUser(userInfoDTO);
           }
           try {
               filterChain.doFilter(servletRequest, servletResponse);
           } finally {
               UserContext.removeUser();
           }
       }
   }
   ```

###### 生成短链接的过程中使用布隆过滤器还是分布式锁

可以将布隆过滤器改为分布式锁，再测试性能

```java
@Override
public ShortLinkCreateRespDTO createShortLinkByLock(ShortLinkCreateReqDTO requestParam) {
    verificationWhitelist(requestParam.getOriginUrl());
    String fullShortUrl;
    // 获取分布式锁
    RLock lock = redissonClient.getLock(SHORT_LINK_CREATE_LOCK_KEY);
    lock.lock();
    try {
        String shortLinkSuffix = generateSuffixByLock(requestParam);
        fullShortUrl = StrBuilder.create(createShortLinkDefaultDomain)
                .append("/")
                .append(shortLinkSuffix)
                .toString();
        ShortLinkDO shortLinkDO = ShortLinkDO.builder()
                .domain(createShortLinkDefaultDomain)
                .originUrl(requestParam.getOriginUrl())
                .gid(requestParam.getGid())
                .createdType(requestParam.getCreatedType())
                .validDateType(requestParam.getValidDateType())
                .validDate(requestParam.getValidDate())
                .describe(requestParam.getDescribe())
                .shortUri(shortLinkSuffix)
                .enableStatus(0)
                .totalPv(0)
                .totalUv(0)
                .totalUip(0)
                .delTime(0L)
                .fullShortUrl(fullShortUrl)
                // 这里记得要将 getFavicon(requestParam.getOriginUrl()) 去掉，性能賊慢
                .favicon(null)
                .build();
        ShortLinkGotoDO linkGotoDO = ShortLinkGotoDO.builder()
                .fullShortUrl(fullShortUrl)
                .gid(requestParam.getGid())
                .build();
        try {
            baseMapper.insert(shortLinkDO);
            shortLinkGotoMapper.insert(linkGotoDO);
        } catch (DuplicateKeyException ex) {
            throw new ServiceException(String.format("短链接：%s 生成重复", fullShortUrl));
        }
        stringRedisTemplate.opsForValue().set(
                String.format(GOTO_SHORT_LINK_KEY, fullShortUrl),
                requestParam.getOriginUrl(),
                LinkUtil.getLinkCacheValidTime(requestParam.getValidDate()), TimeUnit.MILLISECONDS
        );
    } finally {
        lock.unlock();
    }
    return ShortLinkCreateRespDTO.builder()
            .fullShortUrl("http://" + fullShortUrl)
            .originUrl(requestParam.getOriginUrl())
            .gid(requestParam.getGid())
            .build();
}
```

其中generateSuffixByLock方法会差数据库来判断短链接是否存在

```java
private String generateSuffixByLock(ShortLinkCreateReqDTO requestParam) {
    int customGenerateCount = 0;
    String shorUri;
    while (true) {
        if (customGenerateCount > 10) {
            throw new ServiceException("短链接频繁生成，请稍后再试");
        }
        String originUrl = requestParam.getOriginUrl();
        originUrl += UUID.randomUUID().toString();
        shorUri = HashUtil.hashToBase62(originUrl);
        LambdaQueryWrapper<ShortLinkDO> queryWrapper = Wrappers.lambdaQuery(ShortLinkDO.class)
                .eq(ShortLinkDO::getGid, requestParam.getGid())
                .eq(ShortLinkDO::getFullShortUrl, createShortLinkDefaultDomain + "/" + shorUri)
                .eq(ShortLinkDO::getDelFlag, 0);
        ShortLinkDO shortLinkDO = baseMapper.selectOne(queryWrapper);
        if (shortLinkDO == null) {
            break;
        }
        customGenerateCount++;
    }
    return shorUri;
}
```

使用Jmeter对两种方式测试，**大概评估布隆过滤器是分布式锁的 6 倍性能**。理论上说，当并发越高，这个性能差距就越明显。其次，通过分布式锁查询的是 MySQL 数据库，这里还要算上数据库的性能和缓存的差距。

而且，因为我们访问**短链接跳转原始链接接口处理缓存穿透场景**，需要**使用布隆过滤器完成**。所以在这里直接使用是刚好的

这里又有一个问题在于**为什么分布式锁要锁全部的过程**，因为上面也有查库的过程，这是因为考虑在并发场景下，如果查库不设置分布式锁，可能多个线程创建了同一个链接，查库都觉得是不存在的，这个时候就会产生短链接重复问题，导致后面又会重新查库等，消耗性能

###### 消息队列

**不需要延迟队列**

考虑到修改短链接方法中使用的是读写锁，springboot底层使用的tomcat线程池，如果刚开始一来线程池中的两百个线程都被占用了，其中一个线程获取了写锁，其他来的线程只能进入阻塞队列，如果请求多的话，就会引发oom

所以当时的一个思路是当读锁的线程发现有线程持有了写锁，就不会再等，会将任务丢到延迟队列中，就不会有阻塞的耗时

![image-20240623144709067](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240623144709067.png)

后来引入了消息队列，将整个监控都加入了消息队列，而消息队列属于是将任务放到队列中，再匀速的进行读取任务完成，所以延迟队列的意义就不大了，不会再存在oom问题

**使用RocketMQ改造消息队列削峰功能**

**首先引入pom**

```
<rocketmq-spring-boot-starter.version>2.2.3</rocketmq-spring-boot-starter.version>

<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>${rocketmq-spring-boot-starter.version}</version>
</dependency>
```

**编写配置文件**

```
rocketmq:
  name-server: 127.0.0.1:9876
  producer:
    group: short-link_project-service_stats-save_pg
    topic: short-link_project-service_topic
    send-message-timeout: 2000
    retry-times-when-send-failed: 1
    retry-times-when-send-async-failed: 1
  consumer:
    group: short-link_project-service_stats-save_cg
```

**SpringBoot3 适配低版本 RocketMQ 组件库**

创建 META-INF 以及 下属文件夹 spring，在该目录下创建 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件，内容如下：

```
# RocketMQ 2.2.3 version does not adapt to SpringBoot3
org.apache.rocketmq.spring.autoconfigure.RocketMQAutoConfiguration
```

**消息发送者**

短链接监控状态保存消息队列生产者

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class ShortLinkStatsSaveProducer {

    private final RocketMQTemplate rocketMQTemplate;

    @Value("${rocketmq.producer.topic}")
    private String statsSaveTopic;

    /**
     * 发送延迟消费短链接统计
     */
    public void send(Map<String, String> producerMap) {
        String keys = UUID.randomUUID().toString();
        producerMap.put("keys", keys);
        Message<Map<String, String>> build = MessageBuilder
                .withPayload(producerMap)
                .setHeader(MessageConst.PROPERTY_KEYS, keys)
                .build();
        SendResult sendResult;
        try {
            sendResult = rocketMQTemplate.syncSend(statsSaveTopic, build, 2000L);
            log.info("[消息访问统计监控] 消息发送结果：{}，消息ID：{}，消息Keys：{}", sendResult.getSendStatus(), sendResult.getMsgId(), keys);
        } catch (Throwable ex) {
            log.error("[消息访问统计监控] 消息发送失败，消息体：{}", JSON.toJSONString(producerMap), ex);
            // 自定义行为...
        }
    }
}
```

**消息消费者**

```java
@Slf4j
@Component
@RequiredArgsConstructor
@RocketMQMessageListener(
        topic = "${rocketmq.producer.topic}",
        consumerGroup = "${rocketmq.consumer.group}"
)
public class ShortLinkStatsSaveConsumer implements RocketMQListener<Map<String, String>> {

    private final ShortLinkMapper shortLinkMapper;
    private final ShortLinkGotoMapper shortLinkGotoMapper;
    private final RedissonClient redissonClient;
    private final LinkAccessStatsMapper linkAccessStatsMapper;
    private final LinkLocaleStatsMapper linkLocaleStatsMapper;
    private final LinkOsStatsMapper linkOsStatsMapper;
    private final LinkBrowserStatsMapper linkBrowserStatsMapper;
    private final LinkAccessLogsMapper linkAccessLogsMapper;
    private final LinkDeviceStatsMapper linkDeviceStatsMapper;
    private final LinkNetworkStatsMapper linkNetworkStatsMapper;
    private final LinkStatsTodayMapper linkStatsTodayMapper;
    private final MessageQueueIdempotentHandler messageQueueIdempotentHandler;

    @Value("${short-link.stats.locale.amap-key}")
    private String statsLocaleAmapKey;

    @Override
    public void onMessage(Map<String, String> producerMap) {
      //取出key 作为幂等的标识
        String keys = producerMap.get("keys");
        if (!messageQueueIdempotentHandler.isMessageProcessed(keys)) {
            // 判断当前的这个消息流程是否执行完成
            if (messageQueueIdempotentHandler.isAccomplish(keys)) {
                return;
            }
            throw new ServiceException("消息未完成流程，需要消息队列重试");
        }
        try {
            String fullShortUrl = producerMap.get("fullShortUrl");
            if (StrUtil.isNotBlank(fullShortUrl)) {
                String gid = producerMap.get("gid");
                ShortLinkStatsRecordDTO statsRecord = JSON.parseObject(producerMap.get("statsRecord"), ShortLinkStatsRecordDTO.class);
                actualSaveShortLinkStats(fullShortUrl, gid, statsRecord);
            }
        } catch (Throwable ex) {
            log.error("记录短链接监控消费异常", ex);
            throw ex;
        }
        messageQueueIdempotentHandler.setAccomplish(keys);
    }

    public void actualSaveShortLinkStats(String fullShortUrl, String gid, ShortLinkStatsRecordDTO statsRecord) {
        fullShortUrl = Optional.ofNullable(fullShortUrl).orElse(statsRecord.getFullShortUrl());
        RReadWriteLock readWriteLock = redissonClient.getReadWriteLock(String.format(LOCK_GID_UPDATE_KEY, fullShortUrl));
        RLock rLock = readWriteLock.readLock();
        rLock.lock();
        try {
            if (StrUtil.isBlank(gid)) {
                LambdaQueryWrapper<ShortLinkGotoDO> queryWrapper = Wrappers.lambdaQuery(ShortLinkGotoDO.class)
                        .eq(ShortLinkGotoDO::getFullShortUrl, fullShortUrl);
                ShortLinkGotoDO shortLinkGotoDO = shortLinkGotoMapper.selectOne(queryWrapper);
                gid = shortLinkGotoDO.getGid();
            }
            int hour = DateUtil.hour(new Date(), true);
            Week week = DateUtil.dayOfWeekEnum(new Date());
            int weekValue = week.getIso8601Value();
            LinkAccessStatsDO linkAccessStatsDO = LinkAccessStatsDO.builder()
                    .pv(1)
                    .uv(statsRecord.getUvFirstFlag() ? 1 : 0)
                    .uip(statsRecord.getUipFirstFlag() ? 1 : 0)
                    .hour(hour)
                    .weekday(weekValue)
                    .fullShortUrl(fullShortUrl)
                    .gid(gid)
                    .date(new Date())
                    .build();
            linkAccessStatsMapper.shortLinkStats(linkAccessStatsDO);
            Map<String, Object> localeParamMap = new HashMap<>();
            localeParamMap.put("key", statsLocaleAmapKey);
            localeParamMap.put("ip", statsRecord.getRemoteAddr());
            String localeResultStr = HttpUtil.get(AMAP_REMOTE_URL, localeParamMap);
            JSONObject localeResultObj = JSON.parseObject(localeResultStr);
            String infoCode = localeResultObj.getString("infocode");
            String actualProvince = "未知";
            String actualCity = "未知";
            if (StrUtil.isNotBlank(infoCode) && StrUtil.equals(infoCode, "10000")) {
                String province = localeResultObj.getString("province");
                boolean unknownFlag = StrUtil.equals(province, "[]");
                LinkLocaleStatsDO linkLocaleStatsDO = LinkLocaleStatsDO.builder()
                        .province(actualProvince = unknownFlag ? actualProvince : province)
                        .city(actualCity = unknownFlag ? actualCity : localeResultObj.getString("city"))
                        .adcode(unknownFlag ? "未知" : localeResultObj.getString("adcode"))
                        .cnt(1)
                        .fullShortUrl(fullShortUrl)
                        .country("中国")
                        .gid(gid)
                        .date(new Date())
                        .build();
                linkLocaleStatsMapper.shortLinkLocaleState(linkLocaleStatsDO);
            }
            LinkOsStatsDO linkOsStatsDO = LinkOsStatsDO.builder()
                    .os(statsRecord.getOs())
                    .cnt(1)
                    .gid(gid)
                    .fullShortUrl(fullShortUrl)
                    .date(new Date())
                    .build();
            linkOsStatsMapper.shortLinkOsState(linkOsStatsDO);
            LinkBrowserStatsDO linkBrowserStatsDO = LinkBrowserStatsDO.builder()
                    .browser(statsRecord.getBrowser())
                    .cnt(1)
                    .gid(gid)
                    .fullShortUrl(fullShortUrl)
                    .date(new Date())
                    .build();
            linkBrowserStatsMapper.shortLinkBrowserState(linkBrowserStatsDO);
            LinkDeviceStatsDO linkDeviceStatsDO = LinkDeviceStatsDO.builder()
                    .device(statsRecord.getDevice())
                    .cnt(1)
                    .gid(gid)
                    .fullShortUrl(fullShortUrl)
                    .date(new Date())
                    .build();
            linkDeviceStatsMapper.shortLinkDeviceState(linkDeviceStatsDO);
            LinkNetworkStatsDO linkNetworkStatsDO = LinkNetworkStatsDO.builder()
                    .network(statsRecord.getNetwork())
                    .cnt(1)
                    .gid(gid)
                    .fullShortUrl(fullShortUrl)
                    .date(new Date())
                    .build();
            linkNetworkStatsMapper.shortLinkNetworkState(linkNetworkStatsDO);
            LinkAccessLogsDO linkAccessLogsDO = LinkAccessLogsDO.builder()
                    .user(statsRecord.getUv())
                    .ip(statsRecord.getRemoteAddr())
                    .browser(statsRecord.getBrowser())
                    .os(statsRecord.getOs())
                    .network(statsRecord.getNetwork())
                    .device(statsRecord.getDevice())
                    .locale(StrUtil.join("-", "中国", actualProvince, actualCity))
                    .gid(gid)
                    .fullShortUrl(fullShortUrl)
                    .build();
            linkAccessLogsMapper.insert(linkAccessLogsDO);
            shortLinkMapper.incrementStats(gid, fullShortUrl, 1, statsRecord.getUvFirstFlag() ? 1 : 0, statsRecord.getUipFirstFlag() ? 1 : 0);
            LinkStatsTodayDO linkStatsTodayDO = LinkStatsTodayDO.builder()
                    .todayPv(1)
                    .todayUv(statsRecord.getUvFirstFlag() ? 1 : 0)
                    .todayUip(statsRecord.getUipFirstFlag() ? 1 : 0)
                    .gid(gid)
                    .fullShortUrl(fullShortUrl)
                    .date(new Date())
                    .build();
            linkStatsTodayMapper.shortLinkTodayState(linkStatsTodayDO);
        } catch (Throwable ex) {
            log.error("短链接访问量统计异常", ex);
        } finally {
            rLock.unlock();
        }
    }
}
```

删除redis stream configuration

###### 优化：去掉短链接分组监控中的gid字段

问题背景

最开始在监控表中存储了 gid 字段，本意是更好的支持分组查询功能，但是有个问题，那就是短链接的访问量非常大的时候，如果迁移分组，就会涉及到大量的数据变更

考虑到这种场景，我们换了一种实现思路，**gid 不再存储监控表，而是通过短链接表和监控表内联的形势解决 gid 分组查询。**

这样的话，短链接修改就不再需要变更监控表大量 gid，非常好的节省了性能。上面代码中的一大堆修改监控表的代码也就可以删除了。

之前监控表下有gid查询是这样做的

```java
/**
 * 根据分组获取指定日期内PV、UV、UIP数据
 */
@Select("SELECT " +
        "    COUNT(user) AS pv, " +
        "    COUNT(DISTINCT user) AS uv, " +
        "    COUNT(DISTINCT ip) AS uip " +
        "FROM " +
        "    t_link_access_logs " +
        "WHERE " +
        "    gid = #{param.gid} " +
        "    AND create_time BETWEEN #{param.startDate} and #{param.endDate} " +
        "GROUP BY " +
        "    gid;")
LinkAccessStatsDO findPvUvUidStatsByGroup(@Param("param") ShortLinkGroupStatsReqDTO requestParam);
```

将分组标识删除后，就需要短链接表进行内关联查询

```java
/**
 * 根据分组获取指定日期内PV、UV、UIP数据
 */
@Select("SELECT " +
        "    COUNT(tlal.user) AS pv, " +
        "    COUNT(DISTINCT tlal.user) AS uv, " +
        "    COUNT(DISTINCT tlal.ip) AS uip " +
        "FROM " +
        "    t_link tl INNER JOIN " +
        "    t_link_access_logs tlal ON tl.full_short_url = tlal.full_short_url " +
        "WHERE " +
        "    tl.gid = #{param.gid} " +
        "    AND tl.del_flag = '0' " +
        "    AND tl.enable_status = '0' " +
        "    AND tlal.create_time BETWEEN #{param.startDate} and #{param.endDate} " +
        "GROUP BY " +
        "    tl.gid;")
LinkAccessStatsDO findPvUvUidStatsByGroup(@Param("param") ShortLinkGroupStatsReqDTO requestParam);
```

接着将当天查询也取消分表

这是因为之前把监控表的分组标识信息已经删掉，如果说还要保留 `t_link_stats_today` 的分表，那么两个表之前的语句将不再合适，为此，取消了当天监控的分表。

```java
<!-- 分页查询短链接 -->
<select id="pageLink" parameterType="com.nageoffer.shortlink.project.dto.req.ShortLinkPageReqDTO"
        resultType="com.nageoffer.shortlink.project.dao.entity.ShortLinkDO">
    SELECT t.*,
    COALESCE(s.today_pv, 0) AS todayPv,
    COALESCE(s.today_uv, 0) AS todayUv,
    COALESCE(s.today_uip, 0) AS todayUip
    FROM t_link t
    LEFT JOIN t_link_stats_today s ON t.gid = s.gid
    AND t.full_short_url = s.full_short_url
    AND s.date = CURDATE()
    WHERE t.gid = #{gid}
    AND t.enable_status = 0
    AND t.del_flag = 0
    <choose>
        <when test="orderTag == 'todayPv'">
            ORDER BY todayPv DESC
        </when>
        <when test="orderTag == 'todayUv'">
            ORDER BY todayUv DESC
        </when>
        <when test="orderTag == 'todayUip'">
            ORDER BY todayUip DESC
        </when>
        <when test="orderTag == 'totalPv'">
            ORDER BY t.total_pv DESC
        </when>
        <when test="orderTag == 'totalUv'">
            ORDER BY t.total_uv DESC
        </when>
        <when test="orderTag == 'totalUip'">
            ORDER BY t.total_uip DESC
        </when>
        <otherwise>
            ORDER BY t.create_time DESC
        </otherwise>
    </choose>
</select>
```

但这样就有可能导致当天表数据过大问题，为此，可以使用冷热数据分离存储

在 `t_link_stats_today` 表中仅保留需要查询的记录，我们假设保存一个月，然后超过一个月的记录，通过定时任务迁移到 `t_link_stats_today_back` 历史备份表中。`t_link_stats_today` 表中的数据保存的就是热数据，而备份表则是不再查询的冷数据。

###### 不同分组下创建时能保障短链接唯一吗

**问题背景**

假设一个我们有两个分组：abc、bcd。分别创建出了短链接是 /123456，这个时候能保障数据唯一么？

在数据库层面是做了唯一索引兜底的，但因为是按照gid进行了分表的，若创建的两个在一张表上可以保障唯一，但如果说这两个分组不在一张表，唯一索引还能保障短链接唯一么？答案是不能

**解决方案**

考虑到goto表是没有分表的，所以在goto表中full_short_url上加入唯一索引就可以解决这个问题了

###### 用户横向越权问题

对于这个问题，需要将用户名加入数据表中，gid作为每个用户的唯一标识，那这样就要求gid是唯一的，但之前建立的表都是分表的，对于单表gid建立唯一是没用的，所以需要建立一张单表为gid唯一索引作为兜底

**设置分组标识Gid全局唯一**

创建 `t_group_unique` 表单独创建个 gid 唯一索引进行兜底

```
CREATE TABLE `t_group_unique` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `gid` varchar(32) DEFAULT NULL COMMENT '分组标识',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_unique_gid` (`gid`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

之前是通过用户名 username 和分组标识 Gid 一起判断是否重复，现在修改为 Gid 单独控制

**检查用户查询权限**

关于用户查询权限校验有两个说法，一个是 admin、project 都校验，一个是在 project 项目校验，关于这两个优缺点：

- admin、project 都校验：如果用户访问越权，直接在第一层拦截住，就避免了 project 无用调用。缺点是如果用户没有越权，会多一次 admin 检查性能。
- project 校验：如果用户访问越权，会多一层无效调用。优点是用户没有越权，少一次 admin 检查性能。

选择了 project 检验，因为我们做系统面临用户正常访问和不正常访问两种情况和性能挂钩时，尽量选择相信用户。如果是检验 Redis 的话，就倾向于 admin、project 都校验。

在需要检测的业务方法前加入checkGroupBelongToUser进行检验即可

```java
public void checkGroupBelongToUser(String gid) throws ServiceException {
    String username = Optional.ofNullable(UserContext.getUsername())
            .orElseThrow(() -> new ServiceException("用户未登录"));
    LambdaQueryWrapper<GroupDO> queryWrapper = Wrappers.lambdaQuery(GroupDO.class)
            .eq(GroupDO::getGid, gid)
            .eq(GroupDO::getUsername, username);
    List<GroupDO> groupDOList = linkGroupMapper.selectList(queryWrapper);
    if (CollUtil.isEmpty(groupDOList)) {
        throw new ServiceException("用户信息与分组标识不匹配");
    }
}
```

但是这里有个小问题，`UserContext` 是在后管中才有的，我们通过后端系统调用 Peoject，是获取不到用户登录信息，如何获取当前用户？

创建 OpenFeign 请求透传信息 Bean，并获取当前登录用户信息，传递到调用 Peoject 项目的网络请求中

在admin项目中创建一个配置类

```java
@Configuration
public class OpenFeignConfiguration {

    @Bean
    public RequestInterceptor requestInterceptor() {
        return template -> {
            template.header("username", UserContext.getUsername());
            template.header("userId", UserContext.getUserId());
            template.header("realName", UserContext.getRealName());
        };
    }
}
```

远程调用 OpenFeign 接口指定配置类

```java
@FeignClient(
        value = "short-link-project",
        url = "${aggregation.remote-url:}",
        configuration = OpenFeignConfiguration.class
)
public interface ShortLinkActualRemoteService {
}
```

通过这种方案，我们就可以在 Peoject 项目中创建拦截器设置用户上下文

```java
import cn.hutool.core.util.StrUtil;
import jakarta.annotation.Nullable;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

/**
 * 用户信息传输拦截器
 */
@Component
public class UserTransmitFilter implements HandlerInterceptor {

    @Override
    public boolean preHandle(@Nullable HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable Object handler) throws Exception {
        String username = request.getHeader("username");
        if (StrUtil.isNotBlank(username)) {
            String userId = request.getHeader("userId");
            String realName = request.getHeader("realName");
            UserInfoDTO userInfoDTO = new UserInfoDTO(userId, username, realName);
            UserContext.setUser(userInfoDTO);
        }
        return true;
    }

    @Override
    public void afterCompletion(@Nullable HttpServletRequest request, @Nullable HttpServletResponse response, @Nullable Object handler, Exception exception) throws Exception {
        UserContext.removeUser();
    }
}
```

