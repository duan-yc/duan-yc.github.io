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

