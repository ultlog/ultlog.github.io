---
title: 性能测试
toc: true
date: 2020-07-29 22:09:01
tags: 性能
categories: 
---
# 前言
本次性能测试有以下几点说明
- 仅针对ula服务进行查询测试。
- 测试的目标ula服务与elasticsearch服务都为默认配置。即二者启动时均未对连接数/内存等进行调整。详见[系统配置](#系统配置)。
- 仅对ula中日志接口（/api/v1/log）进行测试，该接口内部包含两次elasticsearch查询，加之日常使用极少深度分页，因此该接口在一定程度上能够作为测试基准。
- 本次性能测试的目标为满足五十人左右团队使用时接口不会出现大于1000ms响应。
- 经日常工作经验可知团队内同时使用ultlog的人数大约为1/10 - 1/5 ，因此取极限10绝对并发进行测试。
- 本次测试的总请求次数设置为50*10，即10人各自连续调用服务50次，较大于日常使用ultlog查询日志时的实际情况，因此可以覆盖日常使用的极限值。
- 本次测试的数据量为数日积累，大约在三百万左右。

# 系统配置

在测试之前，待测试ula服务，elasticsearch服务，搭载服务的虚拟机的配置如下

## ula
````shell
jmap -heap {ula pid}
````

````
Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2065694720 (1970.0MB)
   NewSize                  = 42991616 (41.0MB)
   MaxNewSize               = 688390144 (656.5MB)
   OldSize                  = 87031808 (83.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 30932992 (29.5MB)
   used     = 22519952 (21.476699829101562MB)
   free     = 8413040 (8.023300170898438MB)
   72.80237230203919% used
````
更多参数可见[ula_config](https://github.com/ultlog/ula/tree/master/benchmark/ula_config)

## elasticsearch
### 配置
````shell

jhsdb jmap --heap --pid {elasticsearch pid}
````

````
Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 1073741824 (1024.0MB)
   NewSize                  = 348913664 (332.75MB)
   MaxNewSize               = 348913664 (332.75MB)
   OldSize                  = 724828160 (691.25MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 314048512 (299.5MB)
   used     = 248085416 (236.59268951416016MB)
   free     = 65963096 (62.907310485839844MB)
   78.99588965414362% used
````
### 数据量
本次测试的数据量通过如下命令获得：
````shell
curl {elasticsearch host}/ult_index/_count
````
结果：
````json
{"count":34232030,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0}}
````


更多参数可见[ula_config](https://github.com/ultlog/ula/tree/master/benchmark/ela_config)

## linux

````shell
files=$(ls /proc/sys/net/ipv4)
for file in $files
do
  echo "$file" 
  cat "$1"/"$file"
done
````

````
cipso_cache_bucket_size
10
cipso_cache_enable
1
cipso_rbm_optfmt
0
cipso_rbm_strictvalid
1
conf
icmp_echo_ignore_all
0
icmp_echo_ignore_broadcasts
1
icmp_errors_use_inbound_ifaddr
0
````
更多参数可见[ula_config](https://github.com/ultlog/ula/tree/master/benchmark/ipv4_config)

# 测试

## 测试工具
本次测试选择apachbench进行测试，根据[前言](#前言)中所写得到以下命令：
````shell
ab -n 500 -c 10 -g ula_test_output http://{target}/api/v1/log
````

# 测试结果

## 测试报告
````
This is ApacheBench, Version 2.3 <$Revision: 1874286 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking {target} (be patient)


Server Software:        nginx/1.17.1
Server Hostname:        {target}
Server Port:            80

Document Path:          /api/v1/log
Document Length:        4186 bytes

Concurrency Level:      10
Time taken for tests:   6.133 seconds
Complete requests:      500
Failed requests:        0
Total transferred:      2201000 bytes
HTML transferred:       2093000 bytes
Requests per second:    81.53 [#/sec] (mean)
Time per request:       122.660 [ms] (mean)
Time per request:       12.266 [ms] (mean, across all concurrent requests)
Transfer rate:          350.47 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2   1.9      2      15
Processing:    28  105  58.8     91     838
Waiting:       27  102  58.1     88     821
Total:         31  107  58.9     94     839

Percentage of the requests served within a certain time (ms)
  50%     94
  66%    115
  75%    128
  80%    136
  90%    180
  95%    211
  98%    243
  99%    268
 100%    839 (longest request)

````

## 输出结果

````
starttime	seconds	ctime	dtime	ttime	wait
Wed Jul 29 18:39:46 2020	1596019186	3	28	31	27
Wed Jul 29 18:39:47 2020	1596019187	2	35	37	35
Wed Jul 29 18:39:43 2020	1596019183	1	36	37	35
Wed Jul 29 18:39:43 2020	1596019183	5	33	38	30
Wed Jul 29 18:39:43 2020	1596019183	1	37	38	34
Wed Jul 29 18:39:45 2020	1596019185	1	38	39	37
Wed Jul 29 18:39:47 2020	1596019187	5	34	39	34
Wed Jul 29 18:39:46 2020	1596019186	1	39	40	38
Wed Jul 29 18:39:43 2020	1596019183	2	38	40	38
Wed Jul 29 18:39:45 2020	1596019185	9	31	40	28
````
更多可见[ula_test_output](https://github.com/ultlog/ula/tree/master/benchmark/ula_test_output)

# 结论

由测试结果可知本次性能测试可以通过。
