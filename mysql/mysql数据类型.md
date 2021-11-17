## mysql数据类型

### 时间类型（DATE，，TIME，DATETIME，TIMESTEMP）

|           | 范围                                                         | 格式                                   | 其他                             |
| --------- | ------------------------------------------------------------ | -------------------------------------- | -------------------------------- |
| DATETIME  | `'1000-01-01 00:00:00.000000'` to `'9999-12-31 23:59:59.999999'` | *`YYYY-MM-DD hh:mm:ss`*[.*`fraction`*] | contain both date and time parts |
| TIMESTEMP | `'1970-01-01 00:00:01.000000'` UTC to `'2038-01-19 03:14:07.999999'` UTC |                                        |                                  |
| TIME      | '-838:59:59.000000'` to `'838:59:59.000000'                  | *`hh:mm:ss`*[.*`fraction`*]            |                                  |
| DATE      |                                                              |                                        |                                  |

mysql如何存储TIMESTAMP：

MySQL converts `TIMESTAMP` values from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval(This does not occur for other types such as `DATETIME`.) 注意只有TIMESTAMP这样，其他不是的。



可以通过 [`time_zone`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_time_zone) 设置时区。



最好使用系统变量sql_mode设为严格模式，否则的话mysql会将错误的数据格式或者超出范围的数据转换后保存，这样在业务上就会出错了。比如mysql会将'2004-04-31' 转换为'0000-00-00'。



### String类型

 MySQL interprets length specifications in character units（注意`utf8` characters can require up to three bytes per character）



|        |                                                      | 最大存储              | 备注                               |      |
| ------ | ---------------------------------------------------- | --------------------- | ---------------------------------- | ---- |
| char   |                                                      | [0,255]               | mysql的长度都是按照character计算的 |      |
| BINARY | store binary strings rather than nonbinary strings.  |                       |                                    |      |
| Text   | are treated as nonbinary strings (character strings) | 2^16字节（65536字节） |                                    |      |
| ENUM   |                                                      |                       |                                    |      |
| SET    |                                                      |                       |                                    |      |

各种数据类型长度限制： https://dev.mysql.com/doc/refman/5.7/en/storage-requirements.html

```
 a VARCHAR(255) column can hold a string with a maximum length of 255 characters.
```

### Character Sets and Collations（字符集及其比较方式）



- [字符](https://baike.baidu.com/item/字符/4768913)（Character）是各种文字和符号的总称，包括各国家文字、标点符号、图形符号、数字等。字符集（Character set）是多个字符的集合。常见字符集名称：ASCII字符集、GB2312字符集、BIG5字符集、 GB18030字符集、Unicode字符集等。比如简体中文也是个字符集。就是现实中看到的各国文字以及字符。



- 编码：将字符编码为计算机可以识别的二进制数字。（物化现实中的字符）



eg：ASCII字符集由128个[字符](https://baike.baidu.com/item/字符)组成，包括大小写字母、数字0-9、标点符号、非[打印](https://baike.baidu.com/item/打印)字符（换行符、[制表符](https://baike.baidu.com/item/制表符)等4个）以及[控制字符](https://baike.baidu.com/item/控制字符)（退格、响铃等）组成。

​         ASCII字符编码就将A编码为0x41。

同样的简体中文有且仅有一种符号集合（就是我们日常使用的），但是计算机不认识中文啊，只能将其编码为计算机可以识别的二进制数字。目前常用的汉字编码标准主要有 ASCII、[GB2312](https://baike.baidu.com/item/GB2312/483170)、[GBK](https://baike.baidu.com/item/GBK/481954)、[Unicode](https://baike.baidu.com/item/Unicode/750500)等。

- Unicode  全球通用的一项业界标准，包括字符集、编码方案等。Unicode 是为了解决传统的字符编码方案的局限而产生的，它为每种语言中的每个字符设定了统一并且唯一的[二进制编码](https://baike.baidu.com/item/二进制编码/1758517)，以满足跨语言、跨平台进行文本转换、处理的要求。如果把各种文字编码形容为各地的方言，那么[Unicode](https://baike.baidu.com/item/Unicode/750500)就是世界各国合作开发的一种语言。

- UTF-8编码：他是一种编码形式。（由于Unicode比较浪费网络带宽和硬盘，因此为了解决这个问题，就在Unicode的基础上，定义了一套编码规则（将「码位」转换为字节序列的规则（编码/解码 可以理解为 加密/解密 的过程）），这个新的编码规则就是UTF-8，采用1-4个字符进行传输和存储数据。）

[Unicode](https://baike.baidu.com/item/Unicode/750500)是国际组织制定的可以容纳世界上所有文字和符号的[字符编码](https://baike.baidu.com/item/字符编码/8446880)方案。Unicode用数字0-0x10FFFF来映射这些字符，最多可以容纳1114112个字符，或者说有1114112个码位。码位就是可以分配给字符的数字。[UTF-8](https://baike.baidu.com/item/UTF-8/481798)、[UTF-16](https://baike.baidu.com/item/UTF-16/9032026)、[UTF-32](https://baike.baidu.com/item/UTF-32/734460)都是将数字转换到程序数据的编码方案。

```
例如，“汉字”是两个汉语字符，它对应的unicode是0x6c49和0x5b57（\u6c49\u5b57），而编码的程序数据是：
BYTE data_utf8[] = {0xE6, 0xB1, 0x89, 0xE5, 0xAD, 0x97}; // UTF-8编码
WORD data_utf16[] = {0x6c49, 0x5b57}; // UTF-16编码
DWORD data_utf32[] = {0x6c49, 0x5b57}; // UTF-32编码
```

- GBK编码（中国人自己的编码格式，不同于unicode unicode相当于普通话，GBK相当于方言）

  #### UTF8和unicode好GBK之间的关系

  ```
  utf-8--------decode(解码)----->>Unicode类型<<-------decode(解码)-----gbk
  
  utf-8<<--------encode(编码)----->>Unicode类型<<-------encode(编码)----->>gbk
  ```

  ### mysql中的字符集

  MySQL在5.5.3之后增加了这个utf8mb4的编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。好在utf8mb4是utf8的超集，除了将编码改为utf8mb4外不需要做其他转换。当然，为了节省空间，一般情况下使用utf8也就够了。

    那上面说了既然utf8能够存下大部分中文汉字,那为什么还要使用utf8mb4呢? 原来mysql支持的 utf8 编码最大字符长度为 3 字节，如果遇到 4 字节的宽字符就会插入异常了。三个字节的 UTF-8 最大能编码的 Unicode 字符是 0xffff，也就是 Unicode 中的基本多文种平面(BMP)。也就是说，任何不在基本多文本平面的 Unicode字符，都无法使用 Mysql 的 utf8 字符集存储。包括 Emoji 表情(Emoji 是一种特殊的 Unicode 编码，常见于 ios 和 android 手机上)，和很多不常用的汉字，以及任何新增的 Unicode 字符等等(utf8的缺点)。

### Collations（主要用来比较字符的）

