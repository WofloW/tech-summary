尝试用docker-compose以后，就连不上阿里云数据库了，
ping, mongodump, mongo都连不上了
创建了工单后，在对方的指点下找到了原因
route -n
多了一条的路由表

Destination Gateway Genmask     Iface

172.17.0.0  0.0.0.0 255.255.0.0 br-************

阿里云数据库的ip也是172.17.*.*，导致没有从eth0出这部分网络请求

删除这条route的指令

route del -net 172.17.0.0 gw 0.0.0.0 netmask 255.255.0.0 dev br-************

删除后，ping mongodump mongo一切正常

有空研究如何修改docker bridge的地址
