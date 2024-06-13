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

