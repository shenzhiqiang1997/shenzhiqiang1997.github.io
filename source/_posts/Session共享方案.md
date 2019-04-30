---
title: Session共享方案
date: 2018-10-27 00:50:00
tags: 
    - session
    - Java Web
    - 业务场景
categories:
    - Java Web
---
# 场景
考虑这样一个场景：当有多台服务器，由于负载均衡或者分布式业务，使得用户在服务器上A登录之后，要在服务器B上访问用户的会话状态，如何才能够使得用户在访问服务器B时不用重新登录，服务器B直接获取到用户之前的会话状态。  
<!-- more -->
这就需要采用session共享来解决，session共享有多种解决方案。
# 通过cookie共享session  
* 直接将session存放在cookie里，然后在访问多个服务器时，通过cookie中的session信息来进行session的同步。
* 缺点在于不安全，cookie可能被篡改。或者当客户端禁用cookie时，该方法就不适用了。

# 通过MySQL共享session  
1. 基于MySQL的共享服务器  
     * 用一个MySQL服务器存放所有服务器的session，创建和获取session时都访问该MySQL服务器即可。
    * 缺点在于如果该MySQL服务器不可用后，所有服务器的业务都无法进行下去。
2. 基于MySQL的session主从复制
    * 将session表和业务表存放在一起，通过主从复制在多个服务器上进行session的同步。
    * 缺点在于效率不高，会增大数据库的压力，影响整个业务的性能。  

# 通过NFS共享session  
* 建立一个NFS共享服务器，来统一存放所有服务器的session，当创建和获取session时都访问该NFS服务器即可。
* 缺点是当该服务器不可用，整个系统将不能正常运作。另外就是NFS的并发处理能力比较弱，效率不高。

# 通过Web服务器共享session
* 将session写入到服务器内存中，通过服务器提供的同步功能来同步session。
* 缺点在于服务器间的同步的开销是比较大的，并且速度较慢，容易产生性能瓶颈。

# 通过内存数据库共享session
* 将session存放到内存数据库中，多个服务器共享该内存数据库。
* 优点在于内存数据库的过期策略恰好和session的过期对应，另外是存放在内存中的性能很好。