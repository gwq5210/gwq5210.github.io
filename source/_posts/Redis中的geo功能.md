title: Redis中的geo功能
date: 2018-02-03 17:37:12
tags: [Redis, geohash, Redis地理位置功能]
---

# Redis简介
Redis是一个开源的（BSD协议），基于内存的kv存储数据库。它支持很多数据结构。目前Redis还支持部署集群。

从Redis 3.2开始，Redis提供了geo的功能，也就是地理位置相关的命令，今天我们主要来了解这个功能。

<!-- more -->

# geo相关命令简介
跟geo相关的命令有六个：
```
GEOADD key longitude latitude member [longitude latitude member ...]

GEODIST key member1 member2 [unit]

GEOHASH key member [member ...]

GEOPOS key member [member ...]

GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]

GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
```
通过名字我们大概可以知道命令的功能。
* geoadd添加地理位置
* geodist计算地理位置之间的距离
* geohash计算地理位置的geohash(有关geohash的内容可以参考[geohash简介][3])
* geopos获取地理位置
* georadius和georadiusbymember获取某个地理位置附近的所有位置点

下面我们分别对其进行介绍。

# 添加和获取地理位置

## 添加地理位置
我们通过GEOADD来添加位置。
```
GEOADD key longitude latitude member [longitude latitude member ...]
```
可以看到GEOADD每次可以添加多个位置。key是地理位置的集合。

下面是例子，首先添加一个地点清远，后边一次添加多个地点。
```
// 添加一个地点
GEOADD Guangdong-cities 113.2099647 23.593675 Qingyuan

// 添加多个地点
GEOADD Guangdong-cities 113.2278442 23.1255978 Guangzhou 113.106308 23.0088312 Foshan 113.7943267 22.9761989 Dongguan 114.0538788 22.5551603 Shenzhen
```

## 获取地理位置
我们通过GEOPOS来获取对应的地理位置信息。
```
GEOPOS key member [member ...]
```
例如我们通过如下命令来获取刚刚添加进去的地理位置
```
GEOPOS Guangdong-cities Qingyuan Guangzhou Foshan
```
其返回是一个数组，每个数组包含两个元素：经度和维度
```
1) 1) "113.20996731519699"    -- 清远的经度
   2) "23.593675019671288"    -- 清远的纬度
2) 1) "113.22784155607224"    -- 广州的经度
   2) "23.125598202060807"    -- 广州的纬度
3) 1) "113.10631066560745"    -- 佛山的经度
   2) "23.008831202413539"    -- 佛山的纬度
```

# 计算距离和geohash
针对地理位置的操作，Redis提供了计算两点间距离和获取该地理位置geohash的方法

## 计算距离
我们使用GEODIST获取两个位置之间的距离
```
GEODIST key member1 member2 [unit]
```
我们需要制定一个地点集合中的两个地理位置
unit指定返回的距离单位（默认为m），可用值如下
* m 表示单位为米。
* km 表示单位为千米。
* mi 表示单位为英里。
* ft 表示单位为英尺。

例如，计算清远和广州之间的距离
```
GEODIST Guangdong-cities Qingyuan Guangzhou km
"52.094433840356309"    -- 两地相距 52 公里
```

## 计算geohash
相关geohash的资料可以参考[这里][3]

在Redis中我们使用GEOHASH获取一个地点的geohash值
```
GEOHASH key member [member ...]
```
geohash返回一个数组，数组中每个元素都是一个geohash字符串，Redis返回的字符串为11个字符。

例子
```
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEOHASH Sicily Palermo Catania
1) "sqc8b49rny0"
2) "sqdtr74hyu0"
redis> 
```

# 获取附近的点
在我们使用的许多应用中，都有获取附近美食等的能力，Redis也提供了类似的功能，获取附近范围内的地理位置。
有两个命令GEORADIUS和GEORADIUSBYMEMBER，前者用经纬度指定中心点，后者使用存储在Redis中的某个点作为中心点
```
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]

GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
```
其中
* m|km|ft|mi 指定的是计算范围时的单位；
* 如果给定了可选的 WITHCOORD ， 那么命令在返回匹配的位置时会将位置的经纬度一并返回；
* 如果给定了可选的 WITHDIST ， 那么命令在返回匹配的位置时会将位置与中心点之间的距离一并返回；
* 在默认情况下， GEORADIUS 和 GEORADIUSBYMEMBER 的结果是未排序的， ASC 可以让查找结果根据距离从近到远排序， 而 DESC 则可以让查找结果根据从远到近排序；
* COUNT 参数指定要返回的结果数量。

例子
```
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEORADIUS Sicily 15 37 200 km WITHDIST
1) 1) "Palermo"
   2) "190.4424"
2) 1) "Catania"
   2) "56.4413"
redis> GEORADIUS Sicily 15 37 200 km WITHCOORD
1) 1) "Palermo"
   2) 1) "13.36138933897018433"
      2) "38.11555639549629859"
2) 1) "Catania"
   2) 1) "15.08726745843887329"
      2) "37.50266842333162032"

redis> GEOADD Sicily 13.583333 37.316667 "Agrigento"
(integer) 1
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEORADIUSBYMEMBER Sicily Agrigento 100 km
1) "Agrigento"
2) "Palermo"
```

# 小结
Redis提供了地理位置相关的命令，我们可以使用这些特性方便的时间地理位置查找等相关的功能。

# 参考

1) [Redis 命令参考][1]
2) [Redis GEO 特性简介][2]

[1]: http://redisdoc.com/
[2]: http://blog.huangz.me/diary/2015/redis-geo.html
[3]: https://gwq5210.github.io/2018/02/03/geohash%E7%AE%80%E4%BB%8B/