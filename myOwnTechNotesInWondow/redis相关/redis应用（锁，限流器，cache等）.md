理论上讲，和数据相关

### 0 做mybatis的二级缓存

### 1 分布式锁

#### （1）如果仅以效率考虑（2个client拿到锁，业务上可接受，仅影响效率）

​       	 使用单节点lock算法 set key value PX [seconds] https://redis.io/commands/set， 我们现在基本上都用这个方法。

####  (2)  以正确性考虑（2个client都拿到锁将引起严重的问题，业务上不可接受）

​        使用zookeeper（它可以使用ZXID产生一个全局的顺序，进而产生fence，有序性是zookeeper中非常重要的一个特性）

容错的分布式锁：

redlock（对时间做了若干假设）



https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html



按照IP的限流器，



redis是为什么场景设计的？



### 2 缓存

做反向索引 二级索引 以及RPC以及远程db的缓存

### 3 限流器

### 4 社交网络相关

​     微信/微博中点赞关注功能

​     共同好友

### 5 基数统计（会有误差）

只要是计算集合的基数的都可以

​     统计注册 IP 数

​     统计每日访问 IP 数
　 统计页面实时 UV 数
​    统计在线用户数

　统计用户每天搜索不同词条的个数

### 7 布隆过滤器或者bitmap做幂等

### 8 经纬度相关

### 9 消息队列：类似rocketMq的流服务（stream）

10 计数器相关

11 最新列表相关

12 控制库存



见 https://cloud.tencent.com/developer/article/1415674



eg ：使用redis完成一个下面搜索

 ![微信图片_20210831140121](C:\Users\236774\Desktop\微信图片_20210831140121.png)





每个类型手机都用若干属性：品牌，CPU型号，系统，机身存储，运行内存，屏幕尺寸，系统，电池容量，摄像头像素，价格。

（1）使用HashMap保存某型号手机的数据，比如apple11 {apple，Intel，ios， 64G，1G，4.9in.... }

  (2)   建立属性到型号手机的二级索引（使用Set）

​           系统 ----> 手机



（3）搜索的时候，按照二级索引就可以找到符合要求的手机了（使用redis ）

​			使用SINTER key [key ...] 可以求交集。 但是这样的话并不支持区间查询，这种情况必须用Zset支持

（4）对于价格这个需要支持区间查询 + 排序 的放到Zset中，       

```
ZADD priceIndex price ProductId 将产品加入Zset，Score就是价格

ZRANGE key start stop [WITHSCORES] 
	比如ZRANGE priceIndex 1000 1500 WITHSCORES 得到1000-1500元的手机品牌（默认按照分数从低到高）
	
ZREVRANGE priceIndex 0 -1 WITHSCORES     # 按照分数从高到低排列

```