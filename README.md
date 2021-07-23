# croc_remote_file_transfer_tool 
croc，一个简单，高速，安全的跨平台文件及目录树传输工具 
  
前阵子有小伙伴问我有没有这样一个文件及目录树传输工具，具体要求是:   
1.简单  
2.公网对公网，公网对内网，内网对内网全适应  
3.多线程高速   
4.自动断点续传   
5.文件或整个目录树传输  
6.跨平台传输  
7.安全通道传输  
8.加密  
  
于是放狗找了好一阵子，终于找到一个开源工具croc  
https://github.com/schollz/croc  
  
croc这个工具完全满足那个小伙伴的要求，它非常简单，各个平台都只有一个二进制运行文件，  
具体语法也非常简单，原因是它使用了中继服务器，所有关于网络的参数条件都交给中继服务器自动探查及反馈给客户端，  
如果你没有自建中继服务器，那么它会使用croc.schollz.com或croc6.schollz.com作为中继服务器，  
当然如果你觉得传输内容不想泄露，那么你可以自建中继服务器，而且也是简单到打一句命令而已  
  
举例最简单的使用语法为(使用croc.schollz.com或croc6.schollz.com作为中继服务器)  
发送方  
croc send --code 123456 文件名或目录名  
  
接收方(接收内容写在当前目录下)  
croc 123456  
  
用户不需要关心目的地址，来源地址，接收方甚至不需要写接收内容，因为123456这个代码已经确认了接收内容，  
如果中途不知什么原因中断了，或小伙伴主动中断，那么之后随时可以在发送方及接收方分别再打入上面的命令，  
马上就可以开始断点续传了  
  
具体其他使用参数可以查上面的github连接  
  
下面主要是分享一下自建中继服务器的方法及使用  
下面例子使用ubuntu 18或以上版本，这个中继服务器可以运行在linux，windows，macOS等等没有限制,其他系统可以依葫芦画瓢  
转用root用户  
$su  
#cd /tmp  
下载解压及置运行权限  
#wget https://github.com/schollz/croc/releases/download/v9.2.0/croc_9.2.0_Linux-64bit.tar.gz  
#tar xvf croc_9.2.0_Linux-64bit.tar.gz  
#cp ./croc /usr/bin  
#chmod +x /usr/bin/croc  
#cd /  
创建进程服务  
确保是root用户并在根目录,检查是否已经有system目录  
#ls -l /usr/lib/systemd  
  
没有的的话创建system目录  
#mkdir /usr/lib/systemd/system  
  
编辑crocrelay服务文件  
  
#nano /usr/lib/systemd/system/crocrelay.service  
写入以下内容  
  
[Unit]  
Description=crocrelay  
After=network.target  
  
[Service]  
TimeoutStartSec=30  
ExecStart=/usr/bin/croc relay  
ExecStop=/bin/kill $MAINPID  
  
[Install]  
WantedBy=multi-user.target  
  
保存并退出  
  
设置开机启动crocrelay  
#systemctl enable crocrelay  
  
启动crocrelay服务 
#systemctl start crocrelay  
  
检查服务状态  
#systemctl status crocrelay  
  
检查网络连接状态及crocrelay是否有在监听配置9009的端口  
  
#netstat -tnap  
  
至此自己的中继服务器已经搭好了  
  
如果你有一台小鸡是有公网IP的，那么可以把中继服务器架在上面  
而对于有一些小伙伴在各地都有内网互通的，也不想把流量跑到互联网再回来的，那么可以把中继服务器架到内网，  
这样的话就要注意了，实际部署上,如果自建中继服务器在内网，那么所有网段应该是路由可达的，  
否则中继服务器应该拥有所有内网网段的接口地址  
  
下面是一个内网中继服务器的拓扑图，这个例子里面网段10.0.0.x及172.16.0.x没有路由互通，  
意思是PC A不能访问server D：  
     PC A------------server B------------server C------------server D    
10.0.0.20/24       10.0.0.16/24        10.0.0.200/24      
                   172.16.0.30/24      172.16.0.60/24       172.16.0.132/24   
                                       (relay server)  
  
架设中继服务器在内网server C上，server C有两个网段（10.0.0.x及172.16.0.x）  
这时从PC A发送文件并指定中继服务器IP 10.0.0.200(server C)  
croc --relay 10.0.0.200 send --code 123456 filename  
然后屏幕会输出下面的显示，提示接收方要打的命令，如果我们使用公网的IP架中继服务器，  
或内网所有网段都可以路由可达，那么接收方只需按照屏幕输出命令打即可  
Sending 'filename' (16.4 MB)          
Code is: 123456  
On the other computer run  
  
croc --relay 10.0.0.200 123456  
  
从server D接收并指定中继服务器IP 172.25.0.60(server C)，因为server D没有10.0.0.x网段，  
而且172.16.0.x网段与10.0.0.x网段没有互通，所以不能按照发送方屏幕提示的命令IP去打  
croc --relay 172.25.0.6 123456  
  
经过测试，能够用满带宽的最好方法是把所有机器用UDP协议穿透隧道组成大内网，然后在UDP协议下传输croc，  
举例用zerotier,frp等等组网，然后用它们生成的网段进行通信  
  




