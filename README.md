# 网络规划
1. pc1为公网ecs：10.0.7.1/24
2. pc2为美国vps：10.0.7.7/24
3. pc3为本机:	10.0.7.10/24

# 一、先给每台节点配置好hosts并加上公钥
## pc1:
1. ```cd /etc/tinc```
2. ```mkdir -p my-tinc/hosts```
3. ```vim hosts/pc1```
```
按i，粘贴进去  
Address = 公网IP 10070  
Subnet = 10.0.7.1/32
输入“:wq!”保存退出  
```

4. ```tincd -n my-tinc -K4096```  
一直回车完成  

5. ```vim tinc-up```
```
输入i，粘贴进去 
#!/bin/sh  
ip link set $INTERFACE up 
ip addr add 10.0.7.1/24 dev $INTERFACE  
ip route add 10.0.7.0/24 dev $INTERFACE  

输入“:wq!”保存退出
```

6. ```vim tinc-down```
```
输入i，粘贴进去  
#!/bin/sh  
ip route del 10.0.7.0/24 dev $INTERFACE  
ip addr del 10.0.7.1 dev $INTERFACE  
ip link set $INTERFACE down  
  
输入“:wq!”保存退出   
```

7. ```vim tinc.conf```
```
Name = pc1  
AddressFamily = ipv4  
BindToAddress = * 10070  
Interface = tun0  
Device = /dev/net/tun  
ConnectTo = pc1  
ConnectTo = pc2  
ConnectTo = pc3  

输入“:wq!”保存退出  
```

## pc2:
1. cd /etc/tinc
2. mkdir -p my-tinc/hosts
3. vim hosts/pc2
```
按i,粘贴进去  
Subnet = 10.0.7.7/32  

输入“:wq!”保存退出  
```

4. tincd -n my-tinc -K4096
一直回车完成  

5. vim tinc-up
```
输入i，粘贴进去  
#!/bin/sh  
ip link set $INTERFACE up 
ip addr add 10.0.7.7/24 dev $INTERFACE  
ip route add 10.0.7.0/24 dev $INTERFACE   

输入“:wq!”保存退出
```

6. vim tinc-down
```
输入i，粘贴进去
#!/bin/sh  
ip route del 10.0.7.0/24 dev $INTERFACE  
ip addr del 10.0.7.7 dev $INTERFACE  
ip link set $INTERFACE down  

输入“:wq!”保存退出
```
7. vim tinc.conf
```
输入i，粘贴进去  
Name = pc2  
AddressFamily = ipv4  
Interface = tun0  
Device = /dev/net/tun  
ConnectTo = pc1  
ConnectTo = pc2  
ConnectTo = pc3  
输入“:wq!”保存退出  
```


## pc3:
1. cd /etc/tinc
2. mkdir -p my-tinc/hosts
3. vim hosts/pc3
```
按i,粘贴进去  
Subnet = 10.0.7.10/32  

输入“:wq!”保存退出  
```

4. tincd -n my-tinc -K4096
一直回车完成  

5. vim tinc-up
```
输入i，粘贴进去  
#!/bin/sh  
ip link set $INTERFACE up   
ip addr add 10.0.7.10/24 dev $INTERFACE  
ip route add 10.0.7.0/24 dev $INTERFACE  

输入“:wq!”保存退出  
```

6. vim tinc-down
```
输入i，粘贴进去  
#!/bin/sh  
ip route del 10.0.7.0/24 dev $INTERFACE  
ip addr del 10.0.7.10 dev $INTERFACE  
ip link set $INTERFACE down  
输入“:wq!”保存退出  
```
7. vim tinc.conf
```
输入i，粘贴进去  
Name = pc3  
AddressFamily = ipv4  
Interface = tun0  
Device = /dev/net/tun  
ConnectTo = pc1  
ConnectTo = pc2  
ConnectTo = pc3  
输入“:wq!”保存退出  
```

# 二、把各机器的hosts配置传到pc1
## pc2:
```scp hosts/pc2 root@PC1公网IP:/etc/tinc/my-tinc/hosts/pc2  ```

## pc3:
```scp hosts/pc3 root@PC1公网IP:/etc/tinc/my-tinc/hosts/pc3  ```

# 三、各节点把pc1的hosts文件考贝下来
## pc2: 
```scp root@PC1公网IP:/etc/tinc/my-tinc/hosts/* hosts/*  ```

## pc3: 
```scp root@PC1公网IP:/etc/tinc/my-tinc/hosts/* hosts/*  ```

# 四、pc1、pc2、pc3节点分别运行tinc命令
```nohup tincd -n my-tinc -D --debug=5 >> ~/tinc.log 2>&1 &  ```

## pc2 ping pc3:
```ping 10.0.7.10  ```

## pc3 ping pc2
```ping 10.0.7.7  ```

# 附录：
用以下命令查看日志，ctrl+c结束查看日志  
```tail -f ~/tinc.log  ```

查看tinc进程id  
```ps -ef | grep tinc  ```

结束tinc进程，xxxx为上面查看到的进程id  
```kill -9 xxxx  ```





