# OTUS ДЗ№32  Статическая и динамическая маршрутизация, OSPF. #
-----------------------------------------------------------------------
## Домашнее задание ##

1. Поднять три виртуалки
2. Объединить их разными vlan
    * поднять OSPF между машинами на базе Quagga;
    * изобразить ассиметричный роутинг;
    * сделать один из линков "дорогим", но что бы при этом роутинг был симметричным.
-----------------------------------------------------------------------
## Результат ##

Для поднятия стенда можно использовать команду:
```
git clone https://github.com/UtrGrd/otus_hw_32_OSPF && cd otus_hw_32_OSPF && vagrant up
```

### Схема сети: ###
<br/><br/>
 ![Image 1](https://raw.githubusercontent.com/UtrGrd/otus_hw_32_OSPF/da4f9a74f1a52474588260cfe0327b0f2aa80510/ospf.png) <br/><br/>

 
Настроены следующие маршруты:
```
[root@r1 ~]# ip r s
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
172.20.10.0/24 dev eth1 proto kernel scope link src 172.20.10.1 metric 101 
172.20.20.0/24 via 192.168.0.2 dev eth2 proto zebra metric 20 
172.20.30.0/24 via 192.168.200.2 dev eth3 proto zebra metric 20 
192.168.0.0/30 dev eth2 proto kernel scope link src 192.168.0.1 metric 102 
192.168.100.0/30 proto zebra metric 20 
	nexthop via 192.168.0.2 dev eth2 weight 1 
	nexthop via 192.168.200.2 dev eth3 weight 1 
192.168.200.0/30 dev eth3 proto kernel scope link src 192.168.200.1 metric 103
```
```
[root@r2 ~]# ip r s
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
172.20.10.0/24 via 192.168.0.1 dev eth2 proto zebra metric 20 
172.20.20.0/24 dev eth1 proto kernel scope link src 172.20.20.1 metric 101 
172.20.30.0/24 via 192.168.100.1 dev eth3 proto zebra metric 20 
192.168.0.0/30 dev eth2 proto kernel scope link src 192.168.0.2 metric 102 
192.168.100.0/30 dev eth3 proto kernel scope link src 192.168.100.2 metric 103 
192.168.200.0/30 proto zebra metric 20 
	nexthop via 192.168.0.1 dev eth2 weight 1 
	nexthop via 192.168.100.1 dev eth3 weight 
```  
```
[root@r3 ~]# ip r s
default via 10.0.2.2 dev eth0 proto dhcp metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100 
172.20.10.0/24 via 192.168.200.1 dev eth2 proto zebra metric 20 
172.20.20.0/24 via 192.168.100.2 dev eth3 proto zebra metric 20 
172.20.30.0/24 dev eth1 proto kernel scope link src 172.20.30.1 metric 101 
192.168.0.0/30 proto zebra metric 20 
	nexthop via 192.168.200.1 dev eth2 weight 1 
	nexthop via 192.168.100.2 dev eth3 weight 1 
192.168.100.0/30 dev eth3 proto kernel scope link src 192.168.100.1 metric 103 
192.168.200.0/30 dev eth2 proto kernel scope link src 192.168.200.2 metric 102 
```
Роутинг симметричный:
```
[root@r1 ~]# ip a | grep 172
    inet 172.20.10.1/24 brd 172.20.10.255 scope global noprefixroute eth1
[root@r1 ~]# tracepath -n 172.20.20.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  172.20.20.1                                           0.544ms reached
 1:  172.20.20.1                                           0.571ms reached
     Resume: pmtu 1500 hops 1 back 1 
```

Поменяв вес пути в подсети 192.168.0.0/30, связывающей r1 и r2, для интерфейса 192.168.0.1 роутера r1, на , например, 1000, получим ассиметричный роутинг:
```
[root@r1 ~]# vtysh

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

r1# configure  terminal 
r1(config)# interface eth2
r1(config-if)# ip ospf  cost  1000
r1(config-if)# exit
r1(config)# exit
r1# exit
[root@r1 ~]# tracepath -n 172.20.20.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  192.168.200.2                                         0.976ms 
 1:  192.168.200.2                                         0.417ms 
 2:  172.20.20.1                                           1.053ms reached
     Resume: pmtu 1500 hops 2 back 2 
```
Вернуть симметричность роутинга можно, например, повысив вес пути до 1000 в подсети 192.168.100.0/30 для интерфейса 192.168.100.1 роутера r3.
```
[root@r3 ~]# vtysh 

Hello, this is Quagga (version 0.99.22.4).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

r3# configure  terminal  
r3(config)# interface  eth3
r3(config-if)# ip ospf  cost 1000
r3(config-if)# exit
r3(config)# exit
r3# exit
```
```
[root@r1 ~]# tracepath -n 172.20.20.1
 1?: [LOCALHOST]                                         pmtu 1500
 1:  172.20.20.1                                           0.615ms reached
 1:  172.20.20.1                                           0.547ms reached
     Resume: pmtu 1500 hops 1 back 1 
```
