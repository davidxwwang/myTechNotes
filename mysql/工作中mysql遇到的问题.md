## （1）死锁

## 现象：



| 发生时间:2021年10月24日 11:08:50 | 事务 1(已回滚)                                               | 事务 2                                                       |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Session ID                       | 19256392220                                                  | 19256392221                                                  |
| Thread id                        | 8282278                                                      | 8282136                                                      |
| 请求类型                         | updating                                                     | updating                                                     |
| 事务ID                           | 02143000                                                     | 02143001                                                     |
| 涉及表                           | **`paascloud_udc_0_0_v2223`.`acl_object_identity`**          | **`paascloud_udc_0_0_v2223`.`acl_object_identity`****`paascloud_udc_0_0_v2223`.`pc_udc_room_device`** |
| 等待锁                           | index PRIMARY of table `paascloud_udc_0_0_v2223`.`acl_object_identity` trx id 102143000 lock_mode X locks rec but not gap waiting | index key_user_id_pc_udc_room_device of table `paascloud_udc_0_0_v2223`.`pc_udc_room_device` trx id 102143001 lock_mode X locks rec but not gap waiting |
| 等待锁索引名                     | PRIMARY                                                      | key_user_id_pc_udc_room_device                               |
| 等待锁类型                       | X locks rec but not gap waiting                              | X locks rec but not gap waiting                              |
| 持有锁                           |                                                              | index PRIMARY of table `paascloud_udc_0_0_v2223`.`acl_object_identity` trx id 102143001 lock_mode X locks rec but not gap |
| 持有锁索引名                     |                                                              | PRIMARY                                                      |
| 持有锁类型                       | X locks rec but not gap waiting                              | X locks rec but not gap waiting                              |
| 事务SQL                          | update acl_object_identity set parent_object = 1764062 where id = 1762066 | update pc_udc_room_device set deleted = 1 where deleted = 0 and user_id = 267318 and iot_id in ( '2OK6wDKxxxzzRrQPvxiP000000' ) |

该死锁日志

```java
------------------------
LATEST DETECTED DEADLOCK
------------------------
2021-10-24 11:08:50 0x7fa72693d700
/**
* 第一个事务
*/
*** (1) TRANSACTION:
TRANSACTION 102143000, ACTIVE 0 sec starting index read
/** 持有1个锁（locked 1），该事务持有3个行锁（ 3 row lock(s) ）**/
mysql tables in use 1, locked 1
LOCK WAIT 5 lock struct(s), heap size 1136, 3 row lock(s), undo log entries 1
MySQL thread id 8282278, OS thread handle 140356114421504, query id 19256392220 192.168.22.85 gongniu updating
update acl_object_identity set parent_object = 1764062 where id = 1762066
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 296 page no 12617 n bits 128 index PRIMARY of table `paascloud_udc_0_0_v2223`.`acl_object_identity` trx id 102143000 lock_mode X locks rec but not gap waiting
Record lock, heap no 56 PHYSICAL RECORD: n_fields 8; compact format; info bits 32
 0: len 8; hex 00000000001ae312; asc         ;;
 1: len 6; hex 000006169419; asc       ;;
 2: len 7; hex 340000020425e3; asc 4    % ;;
 3: len 8; hex 0000000000000032; asc        2;;
 4: len 26; hex 324f4b3677444b7878787a7a5272515076786950303030303030; asc 2OK6wDKxxxzzRrQPvxiP000000;;
 5: len 8; hex 00000000001aeade; asc         ;;
 6: len 8; hex 0000000000041436; asc        6;;
 7: len 1; hex 81; asc  ;;

/**
* 第二个事务
*/
*** (2) TRANSACTION:
TRANSACTION 102143001, ACTIVE 0 sec starting index read
 /** 持有1个锁（locked 1），该事务持有3个行锁（ 4 row lock(s) ）**/
mysql tables in use 1, locked 1
7 lock struct(s), heap size 1136, 4 row lock(s), undo log entries 2
MySQL thread id 8282136, OS thread handle 140355883489024, query id 19256392221 192.168.22.85 gongniu updating
/**
* 事务二在更新pc_udc_room_device表，update的SQL语句的时候发生死锁，等待`paascloud_udc_0_0_v2223`.`pc_udc_room_device表上的key_user_id_pc_udc_room_device（是user_id的一个普通索引）索引的X锁的释放
*/
update pc_udc_room_device
        set deleted = 1
        where
        deleted = 0
         
         and user_id = 267318
         
        and iot_id in
         (  
            '2OK6wDKxxxzzRrQPvxiP000000'
         )
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 296 page no 12617 n bits 128 index PRIMARY of table `paascloud_udc_0_0_v2223`.`acl_object_identity` trx id 102143001 lock_mode X locks rec but not gap
Record lock, heap no 56 PHYSICAL RECORD: n_fields 8; compact format; info bits 32
 0: len 8; hex 00000000001ae312; asc         ;;
 1: len 6; hex 000006169419; asc       ;;
 2: len 7; hex 340000020425e3; asc 4    % ;;
 3: len 8; hex 0000000000000032; asc        2;;
 4: len 26; hex 324f4b3677444b7878787a7a5272515076786950303030303030; asc 2OK6wDKxxxzzRrQPvxiP000000;;
 5: len 8; hex 00000000001aeade; asc         ;;
 6: len 8; hex 0000000000041436; asc        6;;
 7: len 1; hex 81; asc  ;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
/**
lock type : RECORD LOCKS(锁类型)
space id ： 锁对象的space id
page no ：事务锁定页的数量，如果是表锁，该值为null
index ：锁住的索引
trx id：事务id
lock_mode : 锁的模式
*/
RECORD LOCKS space id 492 page no 3050 n bits 472 index key_user_id_pc_udc_room_device of table `paascloud_udc_0_0_v2223`.`pc_udc_room_device` trx id 102143001 lock_mode X locks rec but not gap waiting
Record lock, heap no 401 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 8; hex 8000000000041436; asc        6;;
 1: len 6; hex 00000006db51; asc      Q;;

/**事务一回滚**/
*** WE ROLL BACK TRANSACTION (1)
```

