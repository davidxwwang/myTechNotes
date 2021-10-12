问题：当有数据更新db时候，如何保证db和redis数据的一致性，即有别的数据更新相同数据，会不会发生脏数据 / 有别的请求请求数据，会不会得到的是过期的数据。



| 步骤 |              |      |              |      |      |
| ---- | ------------ | ---- | ------------ | ---- | ---- |
| (1)  | Del_Redis    |      | Update_DB    |      |      |
| (2)  | Update_Redis |      | Update_DB    |      |      |
| (3)  | Update_DB    |      | Update_Redis |      |      |
| (4)  | Update_DB    |      | Del_Redis    |      |      |
|      |              |      |              |      |      |

假设：

​       （1）单纯的，只更新db中的某一行，redis中key=行主键，value=这行数据。（redis中缓存的数据，有可能是根据更新的行经过计算得到的数据）

​        （2）有两个线程操作。第一个线程只是写入，第二个线程可以读或者也是写入。如果是读：看RW，如果也是写入：看RR

​        （3）更新redis的策略：写入更新：写db时候，更新redis；读更新：读redis，读不到，然后读db，然后更新redis。



### key point

对redis和对db的操作不是原子性的。

注意我们这里讲的数据的一致性，主要讲的是redis的数据一致性，mysql中的数据一直都是OK的。



### 四种要考虑的问题

| RR   | 无需考虑 | 读读 |      |
| ---- | -------- | ---- | ---- |
| RW   | 要考虑   | 读写 |      |
| WW   | 要考虑   | 写写 |      |
|      |          |      |      |

### 

(1) 先删除redis，再更新DB  

| Del_Redis | **Del_Redis_new** | Update_DB_new     | Update_DB         | 会发生覆盖更新，最后的数据是Update_DB的数据，而不是new的数据 |      |
| --------- | ----------------- | ----------------- | ----------------- | ------------------------------------------------------------ | ---- |
| Del_Redis | **Del_Redis_new** | Update_DB         | **Update_DB_new** | 交叉进行                                                     |      |
| Del_Redis | Update_DB         | **Del_Redis_new** | **Update_DB_new** |                                                              |      |

##### RW情况（第二个线程是读）

读发生在Del_Redis和Update_DB之间

​     1）读更新（由读数据触发）：此时读redis，没有数据，然后就去读DB，读到的是旧的数据，已经redis放的都是旧数据了

​     2）写更新（由写入数据触发）：此时读redis，没有数据，返回；到Update_DB 成功后，更新redis

读发生在Update_DB之后

​    1）读更新（由读数据触发）  ：此时读redis，没有数据，然后就去读DB，读到的最新的数据，已经redis放的都是最新数据。

​     2）写更新（由写入数据触发）：此时读redis，是有数据的到(之前的Update_DB 成功后会更新redis

##### WW情况（第二个线程也是写）      

| Del_Redis | **Del_Redis_new** | Update_DB_new     | Update_DB         | 会发生覆盖更新，最后的数据是Update_DB的数据，而不是new的数据 |      |
| --------- | ----------------- | ----------------- | ----------------- | ------------------------------------------------------------ | ---- |
| Del_Redis | **Del_Redis_new** | Update_DB         | **Update_DB_new** | 交叉进行                                                     |      |
| Del_Redis | Update_DB         | **Del_Redis_new** | **Update_DB_new** |                                                              |      |



(2) 先更新redis，再更新DB  （不能这样做）

| update_Redis | **update_Redis_new** | Update_DB_new        | Update_DB         | 危险，如果更新DB失败，还要处理redis |      |
| ------------ | -------------------- | -------------------- | ----------------- | ----------------------------------- | ---- |
| update_Redis | **update_Redis_new** | Update_DB            | **Update_DB_new** | 交叉进行                            |      |
| update_Redis | Update_DB            | **update_Redis_new** | **Update_DB_new** |                                     |      |

(3) 先更新DB，再更新redis  

| update_DB | **update_DB_new** | Update_Redis_new  | Update_Redis         | 危险，如果更新DB失败，还要处理redis |      |
| --------- | ----------------- | ----------------- | -------------------- | ----------------------------------- | ---- |
| update_DB | **update_DB_new** | Update_Redis      | **Update_Redis_new** | 交叉进行                            |      |
| update_DB | Update_Redis      | **update_DB_new** | **Update_Redis_new** |                                     |      |

(4) 先更新DB，再删除redis 

| update_DB | update_DB_new | delete_Redis_NEW  | delete_Redis     | OK             |      |
| --------- | ------------- | ----------------- | ---------------- | -------------- | ---- |
| update_DB | update_DB_new | delete_Redis      | delete_Redis_NEW | 交叉进行，也OK |      |
| update_DB | delete_Redis  | **update_DB_new** | delete_Redis_NEW | OK             |      |





#### linux中的Page Cache与Page回写

https://coolshell.cn/articles/17416.html#Read_Through 还有更新缓存的其他套路：

##### write through

在写入的时候先更新缓存，然后让缓存去更新mysql

##### read throught

在查询操作中更新缓存

##### write behind（write back）

在更新时候只更新缓存，然后让缓存批量的去更新mysql，类似linux中的pageCache算法。（目的还是为了提高IO性能）

###### page cache算法相关

见 https://zhuanlan.zhihu.com/p/71217136

page cache推迟了文件写入后备存储的时间，但是dirty page最终还是要被写回磁盘的。

内核在下面三种情况下会进行会将dirty page写回磁盘：

1. 用户进程调用sync() 和 fsync()系统调用
2. 空闲内存低于特定的阈值（threshold）
3. Dirty数据在内存中驻留的时间超过一个特定的阈值
   





### 如何保证redis里放的都是热点数据

LRU算法（less recently used）最近使用淘汰算法



mySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据?

思路：（1）设置redis最大内存，设置内存淘汰机制是

​			（2）利用Zset给数据评分（每个数据都有ttl），来一次score加1，ttl加长，那些冷数据就被删除了，热数据就留下来了。