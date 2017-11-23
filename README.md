# phpRedis
phpredis的安装和使用基本方法
# phpRedis特性
(1)、持久化数据：redis为了内部的数据安全，会把本身的数据以文件形式保存一份到硬盘，在服务器重启后会自动把硬盘的数据恢复到内存里面，这是和memcache的最大差别，memcache一旦服务器重启或者断电，数据就会清空，而redis会有数据备份，主要AOF和RDB两种持久化方式
(2)、支持主从复制，主机会自动将数据同步到从机，可以进行读写分离
(3)、基于内存的高性能Key-Value存储系统
# Install-phpRedis，php5.5.12环境下安装redis扩展
### 第一步、确定你的php版本，这步十分重要(踩过深坑)
```php
使用phpinfo()函数可以看到你的php环境版本，需要确认以下信息：
  (1)、看Architecture-x64，这个决定你下载的redis安装包多少位，这里是64位，记得不是根据电脑的位数
  链接为：https://github.com/dmajkic/redis/downloads
  (2)、看PHP Extension Buil-API20121212,TS,VC11，这里关注的是TS(线程安全)还是NTS(非线程安全)
  (3)、看php版本到底是5还是7
  根据(2)(3)来下载对应的phpredis安装包，链接为：https://pecl.php.net/package/redis
  根据以上信息下载好对应的redis安装包和phpredis扩展
```
### 第二步、使用cmd安装redis安装包
```php
  (1)、在你想要放redis的盘建一个redis目录
  (2)、使用cmd的cd命令切换到你对应的redis目录下，执行以下命令
    redis-server.exe redis.conf  回车后，此时不要关闭该窗口
  (3)、打开另一个cmd窗口，同样是使用cmd的cd命令切换到你对应的redis目录下，执行以下命令
    redis-cli.exe -h 127.0.0.1 -p 6379 回车后，输入set key test，回车再输入get key，如果可以得到刚刚输入的值就证明redis安装成功了
```    
### 第三步、修改php.ini安装phpredis扩展
```php
  (1)、把下载到的php_redis.dill放到php安装环境目录的ext目录下
  (2)、修改apache目录的bin目录下的php.ini文件，找到extension扩展(有很多extension写在一起的地方)，加上下面命令
    extension=php_redis.dll
```
### 第四步、修改redis.conf文件，开启服务
```php
 把daemonize no 改为daemonize yes
```
### 第五步、重启apache，若再次查看phpinfo的配置信息，能够看到redis扩展就证明安装成功，可以输入以下代码测试
```php
  $redis = new Redis();
  $redis->connect('127.0.0.1', 6379);
  echo "Redis 连接成功";
```
## 注意如果要使用redis服务时，一定要开启，不然会出现以下错误提示
```php
redis went away
```
# phpRedis的基本使用方法
### 连接phpredis扩展
```php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
```
### phpRedis五大基本类型
(1)、string
```php
组合一：$redis->set('string','val');//存值val到set里面
       $val=$redis->get('string');//取出string的值
组合二：$redis->set('string1',4);
       $redis->incr('string1',2);//自增2，输出六
       $val=$redis->get('string1');
       $redis->decr('string',2);//自减2，输出4
       $val=$redis->get('string1');
```
(2)、list(可以实现最新消息排行、消息队列等功能)
```php
组合一：$redis->lPush('list1','A');//依次从左边入队A、B、C
       $redis->lPush('list1','B');
       $redis->lPush('list1','C');
       $val=$redis->rPop('list1');//从右边出队首先是A
```
(3)、set(Set是String类型的无序集合,可以非常方便地实现如共同关注、共同喜好等功能)
```php
组合一：$redis->sAdd('set1','A');
       $redis->sAdd('set1','B');
       $redis->sAdd('set1','B');
       $val=$redis->sCard('set1');//获取set1的个数，相同的元素只取一个，返回值为2
组合二：$val=$redis->sMembers('set1');//数组形式返回set的值，返回值为数组
```
(4)、sort_set(在Set的基础上增加了一个Score的属性，可以排序)
```php
组合一：$redis->zAdd('zset1','60','lilei');//设置三个学生分数存入zset
       $redis->zAdd('zset1','55','linda');
       $redis->zAdd('zset1','80','alin');
       $val=$redis->zRange('zset1','0','-1');//从低到高排名，0代表从0开始，-1代表输出全部
组合二、$val=$redis->zRevRange('zset1','0','-1');//从高到低排名，0代表从0开始，-1代表输出全部
```
(5)、hash(可以存储结构化的信息,如学生的个人信息)
```php
组合一、$redis->hSet('student','name','legend');//设置名为legend的个人信息
       $redis->hSet('student','age','18');
       $redis->hSet('student','height','173cm');
       $val=$redis->hGet('student','name');//取出对应某个信息，返回legend
组合二、$val=$redis->hMGet('student',array('name','age'));//以数组形式返回name和age的信息
```
