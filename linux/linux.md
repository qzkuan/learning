# Linux相关知识



## 常用命令

### find

```sh
# 从当前目录开始查找所有扩展名为.log的文本文件，并找出包含”ERROR”的行
find ./ -type f -name "*.log" | xargs grep -i "error"
```









## 磁盘

- `df -h`：查看磁盘占用情况
- `df -T`：查看所有磁盘的文件系统类型(type)
- `fdisk -l`：查看所有被系统识别的磁盘
- `mount -t type device dir`：挂载device到dir

### **fdisk**

> **命令**用于观察硬盘实体使用情况，可以查看查看新磁盘是否被系统识别

#### 选项

```
-b<分区大小>：指定每个分区的大小；
-l：列出指定的外围设备的分区表状况；
-s<分区编号>：将指定的分区大小输出到标准输出上，单位为区块；
-u：搭配"-l"参数列表，会用分区数目取代柱面数目，来表示每个分区的起始地址；
-v：显示版本信息。
```

用命令fdisk -l查看新磁盘是否被系统识别

```sh
fdisk -l

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048 960335871 960333824 457.9G 83 Linux
/dev/sda2       960337918 976771071  16433154   7.9G  5 Extended
/dev/sda5       960337920 976771071  16433152   7.9G 82 Linux swap / Solaris

Disk /dev/sdb: 2.7 TiB, 3000592982016 bytes, 5860533168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 1AB82561-C5FD-47A9-9A29-31835617EBD3

Device     Start        End    Sectors  Size Type
/dev/sdb1   2048 5860532223 5860530176  2.7T Microsoft basic data
```






### lsblk

> lsblk命令用于列出所有可用块设备的信息，而且还能显示他们之间的依赖关系，但是它不会列出RAM盘的信息。块设备有硬盘，闪存盘，cd-ROM等等。lsblk命令包含在util-linux-ng包中，现在该包改名为util-linux。

#### 选项
```
-a, --all            显示所有设备。
-b, --bytes          以bytes方式显示设备大小。
-d, --nodeps         不显示 slaves 或 holders。
-D, --discard        print discard capabilities。
-e, --exclude <list> 排除设备 (default: RAM disks)。
-f, --fs             显示文件系统信息。
-h, --help           显示帮助信息。
-i, --ascii          use ascii characters only。
-m, --perms          显示权限信息。
-l, --list           使用列表格式显示。
-n, --noheadings     不显示标题。
-o, --output <list>  输出列。
-P, --pairs          使用key="value"格式显示。
-r, --raw            使用原始格式显示。
-t, --topology       显示拓扑结构信息。
````
#### 实例

lsblk命令默认情况下将以树状列出所有块设备。打开终端，并输入以下命令：
```
lsblk

NAME   MAJ:MIN rm   SIZE RO type mountpoint
sda      8:0    0 232.9G  0 disk 
├─sda1   8:1    0  46.6G  0 part /
├─sda2   8:2    0     1K  0 part 
├─sda5   8:5    0   190M  0 part /boot
├─sda6   8:6    0   3.7G  0 part [SWAP]
├─sda7   8:7    0  93.1G  0 part /data
└─sda8   8:8    0  89.2G  0 part /personal
sr0     11:0    1  1024M  0 rom
```
7个栏目名称如下：
```
NAME：这是块设备名。
MAJ:MIN：本栏显示主要和次要设备号。
RM：本栏显示设备是否可移动设备。注意，在本例中设备sdb和sr0的RM值等于1，这说明他们是可移动设备。
SIZE：本栏列出设备的容量大小信息。例如298.1G表明该设备大小为298.1GB，而1K表明该设备大小为1KB。
RO：该项表明设备是否为只读。在本案例中，所有设备的RO值为0，表明他们不是只读的。
TYPE：本栏显示块设备是否是磁盘或磁盘上的一个分区。在本例中，sda和sdb是磁盘，而sr0是只读存储（rom）。
MOUNTPOINT：本栏指出设备挂载的挂载点。
```

### 挂载一个未初始化的磁盘

```sh
1、查看未分配磁盘
   fdisk -l

2、发现有磁盘，路径为/dev/vdb。然后使用fdisk命令进行建立分区
   1）fdisk /dev/vdb
   2）选择“n”
   3）选择“P”
   分区号和扇区都选择默认值。
   4）选择“w”存盘。

3、使用发fdisk -l 查看已经有的分区
4、建好分区后要格式化分区，建立文件系统
    mkfs.ext4 /dev/vdb1
5、这样文件系统就建好了，然后选择一个挂载点挂上就可以了。
   新建挂载点 mkdir /data
    mount /dev/vdb1 /dmdata/   
6、查看挂载是否成功                     
    df -hT

vim /etc/fstab

/dev/vdb1 /dmdata                             ext4    defaults        0 0
```







## 文件传输

### ftp
```
# 1. 首先服务器要安装ftp软件,查看是否已经安装ftp软件下：
which vsftpd

# 2. 查看ftp 服务器状态     
service vsftpd status
# # 3. 启动ftp服务器     
service vsftpd start
# 4. 重启ftp服务器 
service vsftpd restart
# 5. 查看服务有没有启动
netstat -an | grep 21
tcp    0   0 0.0.0.0:21         0.0.0.0:*          LISTEN
# 如果看到以上信息，证明ftp服务已经开启。
# 6.如果需要开启root用户的ftp权限要修改以下两个文件
vi /etc/vsftpd/vsftpd.ftpusers中注释掉root
vi /etc/vsftpd/vsftpd.user_list中也注释掉root
# 然后重新启动ftp服务。
prompt
```

### scp
#### 从本地复制到远程
```
# 文件
scp /home/space/music/1.mp3 root@www.runoob.com:/home/root/others/music 
# 目录
scp -r /home/space/music/ root@www.runoob.com:/home/root/others/ 

```
#### 从远程复制到本地
```
#文件
scp root@www.runoob.com:/home/root/others/music、1.mp3  /home/space/music/
#目录
scp -r www.runoob.com:/home/root/others/ /home/space/music/
```

### sftp
```
名称
     sshd_config - OpenSSH SSH 服务器守护进程配置文件

pwd
     /etc/ssh/sshd_config
 
描述
     sshd(8) 默认从 /etc/ssh/sshd_config 文件(或通过 -f 命令行选项指定的文件)读取配置信息。
     配置文件是由"指令 值"对组成的，每行一个。空行和以'#'开头的行都将被忽略。
     如果值中含有空白符或者其他特殊符号，那么可以通过在两边加上双引号("")进行界定。
     [注意]值是大小写敏感的，但指令是大小写无关的。
     当前所有可以使用的配置指令如下：
     AcceptEnv
             指定客户端发送的哪些环境变量将会被传递到会话环境中。[注意]只有SSH-2协议支持环境变量的传递。
             细节可以参考 ssh_config(5) 中的 SendEnv 配置指令。
             指令的值是空格分隔的变量名列表(其中可以使用'*'和'?'作为通配符)。也可以使用多个 AcceptEnv 达到同样的目的。
             需要注意的是，有些环境变量可能会被用于绕过禁止用户使用的环境变量。由于这个原因，该指令应当小心使用。
             默认是不传递任何环境变量。
     AddressFamily
             指定 sshd(8) 应当使用哪种地址族。取值范围是："any"(默认)、"inet"(仅IPv4)、"inet6"(仅IPv6)。
     AllowGroups
             这个指令后面跟着一串用空格分隔的组名列表(其中可以使用"*"和"?"通配符)。默认允许所有组登录。
             如果使用了这个指令，那么将仅允许这些组中的成员登录，而拒绝其它所有组。
             这里的"组"是指"主组"(primary group)，也就是/etc/passwd文件中指定的组。
             这里只允许使用组的名字而不允许使用GID。相关的 allow/deny 指令按照下列顺序处理：
             DenyUsers, AllowUsers, DenyGroups, AllowGroups
     AllowTcpForwarding
             是否允许TCP转发，默认值为"yes"。
             禁止TCP转发并不能增强安全性，除非禁止了用户对shell的访问，因为用户可以安装他们自己的转发器。
     AllowUsers
             这个指令后面跟着一串用空格分隔的用户名列表(其中可以使用"*"和"?"通配符)。默认允许所有用户登录。
             如果使用了这个指令，那么将仅允许这些用户登录，而拒绝其它所有用户。
             如果指定了 USER@HOST 模式的用户，那么 USER 和 HOST 将同时被检查。
             这里只允许使用用户的名字而不允许使用UID。相关的 allow/deny 指令按照下列顺序处理：
             DenyUsers, AllowUsers, DenyGroups, AllowGroups
     AuthorizedKeysFile
             存放该用户可以用来登录的 RSA/DSA 公钥。
             该指令中可以使用下列根据连接时的实际情况进行展开的符号：
             %% 表示'%'、%h 表示用户的主目录、%u 表示该用户的用户名。
             经过扩展之后的值必须要么是绝对路径，要么是相对于用户主目录的相对路径。
             默认值是".ssh/authorized_keys"。
     Banner
             将这个指令指定的文件中的内容在用户进行认证前显示给远程用户。
             这个特性仅能用于SSH-2，默认什么内容也不显示。"none"表示禁用这个特性。
     ChallengeResponseAuthentication
             是否允许质疑-应答(challenge-response)认证。默认值是"yes"。
             所有 login.conf(5) 中允许的认证方式都被支持。
     Ciphers
             指定SSH-2允许使用的加密算法。多个算法之间使用逗号分隔。可以使用的算法如下：
             "aes128-cbc", "aes192-cbc", "aes256-cbc", "aes128-ctr", "aes192-ctr", "aes256-ctr",
             "3des-cbc", "arcfour128", "arcfour256", "arcfour", "blowfish-cbc", "cast128-cbc"
             默认值是可以使用上述所有算法。
     ClientAliveCountMax
             sshd(8) 在未收到任何客户端回应前最多允许发送多少个"alive"消息。默认值是 3 。
             到达这个上限后，sshd(8) 将强制断开连接、关闭会话。
             需要注意的是，"alive"消息与 TCPKeepAlive 有很大差异。
             "alive"消息是通过加密连接发送的，因此不会被欺骗；而 TCPKeepAlive 却是可以被欺骗的。
             如果 ClientAliveInterval 被设为 15 并且将 ClientAliveCountMax 保持为默认值，
             那么无应答的客户端大约会在45秒后被强制断开。这个指令仅可以用于SSH-2协议。
     ClientAliveInterval
             设置一个以秒记的时长，如果超过这么长时间没有收到客户端的任何数据，
             sshd(8) 将通过安全通道向客户端发送一个"alive"消息，并等候应答。
             默认值 0 表示不发送"alive"消息。这个选项仅对SSH-2有效。
     Compression
             是否对通信数据进行加密，还是延迟到认证成功之后再对通信数据加密。
             可用值："yes", "delayed"(默认), "no"。
     DenyGroups
             这个指令后面跟着一串用空格分隔的组名列表(其中可以使用"*"和"?"通配符)。默认允许所有组登录。
             如果使用了这个指令，那么这些组中的成员将被拒绝登录。
             这里的"组"是指"主组"(primary group)，也就是/etc/passwd文件中指定的组。
             这里只允许使用组的名字而不允许使用GID。相关的 allow/deny 指令按照下列顺序处理：
             DenyUsers, AllowUsers, DenyGroups, AllowGroups
     DenyUsers
             这个指令后面跟着一串用空格分隔的用户名列表(其中可以使用"*"和"?"通配符)。默认允许所有用户登录。
             如果使用了这个指令，那么这些用户将被拒绝登录。
             如果指定了 USER@HOST 模式的用户，那么 USER 和 HOST 将同时被检查。
             这里只允许使用用户的名字而不允许使用UID。相关的 allow/deny 指令按照下列顺序处理：
             DenyUsers, AllowUsers, DenyGroups, AllowGroups
     ForceCommand
             强制执行这里指定的命令而忽略客户端提供的任何命令。这个命令将使用用户的登录shell执行(shell -c)。
             这可以应用于 shell 、命令、子系统的完成，通常用于 Match 块中。
             这个命令最初是在客户端通过 SSH_ORIGINAL_COMMAND 环境变量来支持的。
     GatewayPorts
             是否允许远程主机连接本地的转发端口。默认值是"no"。
             sshd(8) 默认将远程端口转发绑定到loopback地址。这样将阻止其它远程主机连接到转发端口。
             GatewayPorts 指令可以让 sshd 将远程端口转发绑定到非loopback地址，这样就可以允许远程主机连接了。
             "no"表示仅允许本地连接，"yes"表示强制将远程端口转发绑定到统配地址(wildcard address)，
             "clientspecified"表示允许客户端选择将远程端口转发绑定到哪个地址。
     GSSAPIAuthentication
             是否允许使用基于 GSSAPI 的用户认证。默认值为"no"。仅用于SSH-2。
     GSSAPICleanupCredentials
             是否在用户退出登录后自动销毁用户凭证缓存。默认值是"yes"。仅用于SSH-2。
     HostbasedAuthentication
             这个指令与 RhostsRSAAuthentication 类似，但是仅可以用于SSH-2。推荐使用默认值"no"。
             推荐使用默认值"no"禁止这种不安全的认证方式。
     HostbasedUsesNameFromPacketOnly
             在开启 HostbasedAuthentication 的情况下，
             指定服务器在使用 ~/.shosts ~/.rhosts /etc/hosts.equiv 进行远程主机名匹配时，是否进行反向域名查询。
             "yes"表示 sshd(8) 信任客户端提供的主机名而不进行反向查询。默认值是"no"。
     HostKey
             主机私钥文件的位置。如果权限不对，sshd(8) 可能会拒绝启动。
             SSH-1默认是 /etc/ssh/ssh_host_key 。
             SSH-2默认是 /etc/ssh/ssh_host_rsa_key 和 /etc/ssh/ssh_host_dsa_key 。
             一台主机可以拥有多个不同的私钥。"rsa1"仅用于SSH-1，"dsa"和"rsa"仅用于SSH-2。
     IgnoreRhosts
             是否在 RhostsRSAAuthentication 或 HostbasedAuthentication 过程中忽略 .rhosts 和 .shosts 文件。
             不过 /etc/hosts.equiv 和 /etc/shosts.equiv 仍将被使用。推荐设为默认值"yes"。
     IgnoreUserKnownHosts
             是否在 RhostsRSAAuthentication 或 HostbasedAuthentication 过程中忽略用户的 ~/.ssh/known_hosts 文件。
             默认值是"no"。为了提高安全性，可以设为"yes"。
     KerberosAuthentication
             是否要求用户为 PasswordAuthentication 提供的密码必须通过 Kerberos KDC 认证，也就是是否使用Kerberos认证。
             要使用Kerberos认证，服务器需要一个可以校验 KDC identity 的 Kerberos servtab 。默认值是"no"。
     KerberosGetAFSToken
             如果使用了 AFS 并且该用户有一个 Kerberos 5 TGT，那么开启该指令后，
             将会在访问用户的家目录前尝试获取一个 AFS token 。默认为"no"。
     KerberosOrLocalPasswd
             如果 Kerberos 密码认证失败，那么该密码还将要通过其它的认证机制(比如 /etc/passwd)。
             默认值为"yes"。
     KerberosTicketCleanup
             是否在用户退出登录后自动销毁用户的 ticket 。默认值是"yes"。
     KeyRegenerationInterval
             在SSH-1协议下，短命的服务器密钥将以此指令设置的时间为周期(秒)，不断重新生成。
             这个机制可以尽量减小密钥丢失或者黑客攻击造成的损失。
             设为 0 表示永不重新生成，默认为 3600(秒)。
     ListenAddress
             指定 sshd(8) 监听的网络地址，默认监听所有地址。可以使用下面的格式：
                   ListenAddress host|IPv4_addr|IPv6_addr
                   ListenAddress host|IPv4_addr:port
                   ListenAddress [host|IPv6_addr]:port
             如果未指定 port ，那么将使用 Port 指令的值。
             可以使用多个 ListenAddress 指令监听多个地址。
     LoginGraceTime
             限制用户必须在指定的时限内认证成功，0 表示无限制。默认值是 120 秒。
     LogLevel
             指定 sshd(8) 的日志等级(详细程度)。可用值如下：
             QUIET, FATAL, ERROR, INFO(默认), VERBOSE, DEBUG, DEBUG1, DEBUG2, DEBUG3
             DEBUG 与 DEBUG1 等价；DEBUG2 和 DEBUG3 则分别指定了更详细、更罗嗦的日志输出。
             比 DEBUG 更详细的日志可能会泄漏用户的敏感信息，因此反对使用。
     MACs
             指定允许在SSH-2中使用哪些消息摘要算法来进行数据校验。
             可以使用逗号分隔的列表来指定允许使用多个算法。默认值(包含所有可以使用的算法)是：
             hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd160,hmac-sha1-96,hmac-md5-96
     Match
             引入一个条件块。块的结尾标志是另一个 Match 指令或者文件结尾。
             如果 Match 行上指定的条件都满足，那么随后的指令将覆盖全局配置中的指令。
             Match 的值是一个或多个"条件-模式"对。可用的"条件"是：User, Group, Host, Address 。
             只有下列指令可以在 Match 块中使用：AllowTcpForwarding, Banner,
             ForceCommand, GatewayPorts, GSSApiAuthentication,
             KbdInteractiveAuthentication, KerberosAuthentication,
             PasswordAuthentication, PermitOpen, PermitRootLogin,
             RhostsRSAAuthentication, RSAAuthentication, X11DisplayOffset,
             X11Forwarding, X11UseLocalHost
     MaxAuthTries
             指定每个连接最大允许的认证次数。默认值是 6 。
             如果失败认证的次数超过这个数值的一半，连接将被强制断开，且会生成额外的失败日志消息。
     MaxStartups
             最大允许保持多少个未认证的连接。默认值是 10 。
             到达限制后，将不再接受新连接，除非先前的连接认证成功或超出 LoginGraceTime 的限制。
     PasswordAuthentication
             是否允许使用基于密码的认证。默认为"yes"。
     PermitEmptyPasswords
             是否允许密码为空的用户远程登录。默认为"no"。
     PermitOpen
             指定TCP端口转发允许的目的地，可以使用空格分隔多个转发目标。默认允许所有转发请求。
             合法的指令格式如下：
                   PermitOpen host:port
                   PermitOpen IPv4_addr:port
                   PermitOpen [IPv6_addr]:port
             "any"可以用于移除所有限制并允许一切转发请求。
     PermitRootLogin
             是否允许 root 登录。可用值如下：
             "yes"(默认) 表示允许。"no"表示禁止。
             "without-password"表示禁止使用密码认证登录。
             "forced-commands-only"表示只有在指定了 command 选项的情况下才允许使用公钥认证登录。
                                   同时其它认证方法全部被禁止。这个值常用于做远程备份之类的事情。
     PermitTunnel
             是否允许 tun(4) 设备转发。可用值如下：
             "yes", "point-to-point"(layer 3), "ethernet"(layer 2), "no"(默认)。
             "yes"同时蕴含着"point-to-point"和"ethernet"。
     PermitUserEnvironment
             指定是否允许 sshd(8) 处理 ~/.ssh/environment 以及 ~/.ssh/authorized_keys 中的 environment= 选项。
             默认值是"no"。如果设为"yes"可能会导致用户有机会使用某些机制(比如 LD_PRELOAD)绕过访问控制，造成安全漏洞。
     PidFile
             指定在哪个文件中存放SSH守护进程的进程号，默认为 /var/run/sshd.pid 文件。
     Port
             指定 sshd(8) 守护进程监听的端口号，默认为 22 。可以使用多条指令监听多个端口。
             默认将在本机的所有网络接口上监听，但是可以通过 ListenAddress 指定只在某个特定的接口上监听。
     PrintLastLog
             指定 sshd(8) 是否在每一次交互式登录时打印最后一位用户的登录时间。默认值是"yes"。
     PrintMotd
             指定 sshd(8) 是否在每一次交互式登录时打印 /etc/motd 文件的内容。默认值是"yes"。
     Protocol
             指定 sshd(8) 支持的SSH协议的版本号。
             '1'和'2'表示仅仅支持SSH-1和SSH-2协议。"2,1"表示同时支持SSH-1和SSH-2协议。
     PubkeyAuthentication
             是否允许公钥认证。仅可以用于SSH-2。默认值为"yes"。
     RhostsRSAAuthentication
             是否使用强可信主机认证(通过检查远程主机名和关联的用户名进行认证)。仅用于SSH-1。
             这是通过在RSA认证成功后再检查 ~/.rhosts 或 /etc/hosts.equiv 进行认证的。
             出于安全考虑，建议使用默认值"no"。
     RSAAuthentication
             是否允许使用纯 RSA 公钥认证。仅用于SSH-1。默认值是"yes"。
     ServerKeyBits
             指定临时服务器密钥的长度。仅用于SSH-1。默认值是 768(位)。最小值是 512 。
     StrictModes
             指定是否要求 sshd(8) 在接受连接请求前对用户主目录和相关的配置文件进行宿主和权限检查。
             强烈建议使用默认值"yes"来预防可能出现的低级错误。
     Subsystem
             配置一个外部子系统(例如，一个文件传输守护进程)。仅用于SSH-2协议。
             值是一个子系统的名字和对应的命令行(含选项和参数)。比如"sft /bin/sftp-server"。
     SyslogFacility
             指定 sshd(8) 将日志消息通过哪个日志子系统(facility)发送。有效值是：
             DAEMON, USER, AUTH(默认), LOCAL0, LOCAL1, LOCAL2, LOCAL3, LOCAL4, LOCAL5, LOCAL6, LOCAL7
     TCPKeepAlive
             指定系统是否向客户端发送 TCP keepalive 消息。默认值是"yes"。
             这种消息可以检测到死连接、连接不当关闭、客户端崩溃等异常。
             可以设为"no"关闭这个特性。
     UseDNS
             指定 sshd(8) 是否应该对远程主机名进行反向解析，以检查此主机名是否与其IP地址真实对应。默认值为"yes"。
     UseLogin
             是否在交互式会话的登录过程中使用 login(1) 。默认值是"no"。
             如果开启此指令，那么 X11Forwarding 将会被禁止，因为 login(1) 不知道如何处理 xauth(1) cookies 。
             需要注意的是，login(1) 是禁止用于远程执行命令的。
             如果指定了 UsePrivilegeSeparation ，那么它将在认证完成后被禁用。
     UsePrivilegeSeparation
             是否让 sshd(8) 通过创建非特权子进程处理接入请求的方法来进行权限分离。默认值是"yes"。
             认证成功后，将以该认证用户的身份创建另一个子进程。
             这样做的目的是为了防止通过有缺陷的子进程提升权限，从而使系统更加安全。
     X11DisplayOffset
             指定 sshd(8) X11 转发的第一个可用的显示区(display)数字。默认值是 10 。
             这个可以用于防止 sshd 占用了真实的 X11 服务器显示区，从而发生混淆。
     X11Forwarding
             是否允许进行 X11 转发。默认值是"no"，设为"yes"表示允许。
             如果允许X11转发并且sshd(8)代理的显示区被配置为在含有通配符的地址(X11UseLocalhost)上监听。
             那么将可能有额外的信息被泄漏。由于使用X11转发的可能带来的风险，此指令默认值为"no"。
             需要注意的是，禁止X11转发并不能禁止用户转发X11通信，因为用户可以安装他们自己的转发器。
             如果启用了 UseLogin ，那么X11转发将被自动禁止。
     X11UseLocalhost
             sshd(8) 是否应当将X11转发服务器绑定到本地loopback地址。默认值是"yes"。
             sshd 默认将转发服务器绑定到本地loopback地址并将 DISPLAY 环境变量的主机名部分设为"localhost"。
             这可以防止远程主机连接到 proxy display 。不过某些老旧的X11客户端不能在此配置下正常工作。
             为了兼容这些老旧的X11客户端，你可以设为"no"。
     XAuthLocation
             指定 xauth(1) 程序的绝对路径。默认值是 /usr/X11R6/bin/xauth
时间格式
     在 sshd(8) 命令行参数和配置文件中使用的时间值可以通过下面的格式指定：time[qualifier] 。
     其中的 time 是一个正整数，而 qualifier 可以是下列单位之一：
           <无>    秒
           s | S   秒
           m | M   分钟
           h | H   小时
           d | D   天
           w | W   星期
     可以通过指定多个数值来累加时间，比如：
           1h30m   1 小时 30 分钟 (90 分钟)
文件
     /etc/ssh/sshd_config
             sshd(8) 的主配置文件。这个文件的宿主应当是root，权限最大可以是"644"。
```

## Redhat更换yum源

> 1. 卸载Redhat自带的yum包
```sh
[root@localhost ~]# rpm -qa |grep yum
yum-rhn-plugin-2.0.1-10.el7.noarch
yum-metadata-parser-1.1.4-10.el7.x86_64
yum-3.4.3-161.el7.noarch
[root@localhost ~]# rpm -e yum-rhn-plugin-2.0.1-10.el7.noarch
# error: Failed dependencies: yum-rhn-plugin >= 1.6.4-1 is needed by (installed) rhn-check-2.0.2-24.el7.x86_64
# 解决:最后加上 --nodeps
[root@localhost ~]# rpm -e yum-rhn-plugin-2.0.1-10.el7.noarch --nodeps
[root@localhost ~]# rpm -qa |grep yum                        
yum-metadata-parser-1.1.4-10.el7.x86_64
yum-3.4.3-161.el7.noarch
[root@localhost ~]# rpm -e yum-metadata-parser-1.1.4-10.el7.x86_64 --nodeps
[root@localhost ~]# rpm -e yum-3.4.3-161.el7.noarch --nodeps
[root@localhost ~]# rpm -qa |grep yum
[root@localhost ~]# 
```

> 2. 下载centeros的yum包并ftp传到/opt/tools/目录下

[aliyum镜像地址](https://mirrors.aliyun.com/centos/7/os/x86_64/Packages/)

[网易镜像地址](http://mirrors.163.com/centos/7/os/x86_64/Packages/)

> 3. 安装yum

```sh
[root@localhost ~]# cd /opt/tools
[root@localhost tools]# ll
total 1572
-rw-r--r--. 1 root root  111048 May 27 13:52 python-urlgrabber-3.10-10.el7.noarch.rpm
-rw-r--r--. 1 root root 1298672 May 27 13:52 yum-3.4.3-167.el7.centos.noarch.rpm
-rw-r--r--. 1 root root   28348 May 27 13:51 yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
-rw-r--r--. 1 root root   35004 May 27 13:51 yum-plugin-fastestmirror-1.1.31-53.el7.noarch.rpm
-rw-r--r--. 1 root root  124628 May 27 13:52 yum-utils-1.1.31-53.el7.noarch.rpm

[root@localhost tools]# rpm -ivh python-urlgrabber-3.10-10.el7.noarch.rpm --force --nodeps
warning: python-urlgrabber-3.10-10.el7.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:python-urlgrabber-3.10-10.el7    ################################# [100%]
[root@localhost tools]# 

[root@localhost tools]# rpm -ivh yum* --force --nodeps
warning: yum-3.4.3-167.el7.centos.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:yum-metadata-parser-1.1.4-10.el7 ################################# [ 25%]
   2:yum-plugin-fastestmirror-1.1.31-5################################# [ 50%]
   3:yum-3.4.3-167.el7.centos         ################################# [ 75%]
   4:yum-utils-1.1.31-53.el7          ################################# [100%]
   
# 检查是否安装成功
[root@localhost tools]# rpm -qa |grep yum
yum-metadata-parser-1.1.4-10.el7.x86_64
yum-3.4.3-167.el7.centos.noarch
yum-plugin-fastestmirror-1.1.31-53.el7.noarch
yum-utils-1.1.31-53.el7.noarch


```

> 4. 修改repo
```sh
# 下载源文件
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 需要把Centos-7.repo文件中的$releasever全部替换为7
67  cd /etc/yum.repos.d/
68  ls
69  vi CentOS-Base.repo
# 在vim中执行:%s/$releasever/7/g快速替换。保存退出。
```

> 5.清空重载yum

```sh
yum clean all
yum update
# 最后查看一下yum列表
yum list
```


## 用户相关
```sh
# 修改文件所属组群——chgrp
chgrp  组群  文件名/目录 
# 修改文件拥有者——chown
chown [-R] 账号名称      文件/目录
chown [-R] 账号名称:组群  文件/目录
# 改变文件权限——chmod
# -rw-r--r-- 所表示含义，linux为每一个权限分配一个固定的数字：
 r： 4（读权限）
 w： 2（写权限）
 x： 1（执行权限）
-rw-r--r--  1 myy groupa 0 Sep 26 06:07 filed
第一组（user）：rw- = 4+2+0 = 6
第二组（group）：r-- = 4+0+0 = 4
第三组（others）：r-- = 4+0+0 = 4
```
>chgrp，chown，chmod这些命令默认的情况下只有root有权限执行，大家有时可能会用普通账户去修改文件权限，linux会提示你没有这个权限。`Operation not permitted`

## RPM
### RPM常用命令：

* -i   安装软件包
* --nodeps   不验证软件包的依赖
* -v  可视化，提供更多的详细信息的输出
* -h  显示安装进度
* 另外的常用的附带参数为：
* --force 强制安装，即使覆盖其他包的文件也要安装
* -a 查询所有已经安装的软件包
*  -f 查询 文件所属于的软件包
*  -q 查询软件包（通常用来看下还未安装的软件包）
*  -l 显示软件包的文件列表
*  -d 显示被标注为文档的文件列表
*  -c 显示被标注为配置文件的文件列表，最后这两个用的很少了

## Linux之ln命令
### 一、介绍
> ln命令用于将一个文件创建链接,链接分为软链接(类似于windows系统中的快捷方式)和硬链接(相当于对源文件copy,程序或命令对该文件block的另一个访问路口)，命令默认使用硬链接。

### 二、使用方法

* 语法：ln [选项][文件]
* 选项：-s 对源文件创建软链接


### 三、案例：
> 1.对文件创建软链接

```sh
[root@ping ~]# ln -s /root/student.sql /root/db/ln.sql
[root@ping ~]# ls -lh db/ln.sql
lrwxrwxrwx 1 root root 17 2月  23 15:31 db/ln.sql -> /root/student.sql
```
> 2.对目录创建软链接

```sh
[root@ping ~]# ln -s db data
[root@ping ~]# ll -h data/
lrwxrwxrwx 1 root root 17 2月  23 15:31 ln.sql -> /root/student.sql
[root@ping ~]# ln student.sql db/
```
> 3.对文件创建硬链接

```sh
[root@ping ~]# ln student.sql db/
[root@ping ~]# ls -lh db/
lrwxrwxrwx 1 root root   17 2月  23 15:31 ln.sql -> /root/student.sql
-rw-r--r-- 2 root root 2.9K 2月  12 10:17 student.sql
```

### 四、软、硬链接说明　
- 软链接：不可以删除源文件，删除源文件导致链接文件找不到，出现文件红色闪烁
+ 硬链接：可以删除源文件，链接文件可以正常打开

## 安装jdk

### 方式一：yum方式下载安装
> 1、查找java相关的列表
```sh
yum -y list java*
# 或者
yum search jdk
```
>2、安装jdk
```sh
yum install java-1.8.0-openjdk.x86_64
```
> 3、完成安装后验证
```sh
java -version
```
> 4、通过yum安装的默认路径为：/usr/lib/jvm

> 5、将jdk的安装路径加入到JAVA_HOME
```sh
vi /etc/profile
# 在文件最后加入：
# set java environment
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME CLASSPATH PATH

#修改/etc/profile之后让其生效
. /etc/profile #（注意 . 之后应有一个空格）
```
方式二、官网下载jdk，ftp上传服务器解压安装
1、进入 Oracle 官方网站 下载合适的 JDK 版本，准备安装。
注意：这里需要下载 Linux 版本。这里以jdk-8u151-linux-x64.tar.gz为例，你下载的文件可能不是这个版本，这没关系，只要后缀(.tar.gz)一致即可。

2、创建目录

在/usr/目录下创建java目录，

mkdir /usr/local/java
cd /usr/local/java
把下载的文件 jdk-8u151-linux-x64.tar.gz 放在/usr/local/java/目录下。

3. 解压 JDK

tar -zxvf jdk-8u151-linux-x64.tar.gz

4. 设置环境变量

修改 vi /etc/profile

在 profile 文件中添加如下内容并保存：

set java environment
JAVA_HOME=/usr/local/java/jdk1.8.0_151        
JRE_HOME=/usr/local/java/jdk1.8.0_151/jre     
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH


注意：其中 JAVA_HOME， JRE_HOME 请根据自己的实际安装路径及 JDK 版本配置。

让修改生效：

source /etc/profile

5. 测试
java -version

显示 java 版本信息，则说明 JDK 安装成功


## 防火墙

> firewall-cmd 是firewalld服务的一个命令行客户端，提供了对防火墙规则的增删查改。firewalld自身并不具备防火墙的功能。它和iptables一样需要通过内核netfilter来实现防火墙的功能，相对于iptables来说firewall-cmd使用起来更简单。

### 防火墙的启动停止(centos7以上)
```
 systemctl  status  firewalld.service  #查看防火墙状态
 
 systemctl start firewalld.service     #启动防火墙
 
 systemctl stop firewalld.service      #停止防火墙
 
 systemctl restart firewalld.service   #重新启动防火墙
 
 systemctl enable firewalld.service    #设置防火墙开机自启动
 
 systemctl disable firewalld.service   #禁止防火墙开机自启动
```
 ### firewall-cmd
 ```
 1、 查看开放的端口
firewall-cmd  --list-ports 
2、开放端口
#将端口添加到域 public，重启防火墙后失效
firewall-cmd  --zone=public  --add-port=3456/tcp  # tcp 可以是 {'tcp'|'udp'|'sctp'|'dccp'}中的一个

#将端口添加到域 public，想要永久生效，添加 --permanent 参数
 firewall-cmd  --zone=public  --add-port=3456/tcp --permanent 
3、关闭开放的端口
 #将端口从域 public 中删除，重启防火墙后失效
 firewall-cmd  --zone=public  --remove-port=3456/tcp
 
 #将端口从域 public 中删除，想要永久生效，添加 --permanent 参数 
 firewall-cmd  --zone=public  --remove-port=3456/tcp --permanent 
 ```


```
# 开启防火墙 
systemctl start firewalld
# 关闭防火墙 
systemctl stop firewalld
# 重启防火墙 
systemctl restart firewalld
# 防火墙状态 
systemctl status firewalld

# 打开虚拟机防火墙对应端口
firewall-cmd --state //查看运行状态
# 开放5601的端口
firewall-cmd --zone=public --add-port=5601/tcp --permanent
# 命令含义：
# --zone #作用域
# --add-port=80/tcp  #添加端口，格式为：端口/通讯协议
# --permanent 永久生效，没有此参数重启后失效

# 重载生效刚才的端口设置
firewall-cmd --reload

firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=192.168.0.1/2 port=80 port protocol=tcp accept'
```

## 打包
```
# 命令格式：tar -zxvf 【压缩包文件名.tar.gz】 -C  【路径】/
# 注释：解压.tar.gz格式到指定的目录下
tar -zxvf japan.tar.gz -C /tmp/

# 压缩.tar.gz格式到指定目录下
# 命令格式：tar -zcvf 【目录】/ 【压缩包文件名.tar.gz】【源文件】
# 注意：一次压缩多个文件直接在源文件后用空格格开即可
tar -zcvf /tmp/test.tar.gz japan/
# 将文件夹 abc 进行压缩时，排除1.txt
tar --exclude=abc/1.txt -zcvf abc.tar.gz abc

# 列出压缩文件内容
tar -ztvf test.tar.gz 



```


## crontab定时任务
****crontab简介****
> crontab就是一个自定义定时器。

****crontab配置文件****

- 其一：`/var/spool/cron/`
该目录下存放的是每个用户（包括root）的crontab任务，文件名以用户名命名
- 其二：`/etc/cron.d/`
这个目录用来存放任何要执行的crontab文件或脚本。
****crontab时间说明****
```
# .---------------- minute (0 - 59) 
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ... 
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7)  OR
#sun,mon,tue,wed,thu,fri,sat 
# |  |  |  |  |
# *  *  *  *  *  command to be executed
minute：代表一小时内的第几分，范围 0-59。
hour：代表一天中的第几小时，范围 0-23。
mday：代表一个月中的第几天，范围 1-31。
month：代表一年中第几个月，范围 1-12。
wday：代表星期几，范围 0-7 (0及7都是星期天)。
who：要使用什么身份执行该指令，当您使用 crontab -e 时，不必加此字段。
command：所要执行的指令。
```
****crontab服务状态****
```
sudo service crond start     #启动服务
sudo service crond stop      #关闭服务
sudo service crond restart   #重启服务
sudo service crond reload    #重新载入配置
sudo service crond status    #查看服务状态
```
****crontab命令****
重新指定crontab定时任务列表文件
```
crontab $filepath
```
查看crontab定时任务
```
crontab -l
```
编辑定时任务【删除-添加-修改】
```
crontab -e
```
添加定时任务【推荐】
```
Step-One : 编辑任务脚本【分目录存放】【ex: backup.sh】
Step-Two : 编辑定时文件【命名规则:backup.cron】
Step-Three : crontab命令添加到系统crontab backup.cron
Step-Four : 查看crontab列表 crontab -l
```

****crontab时间举例****
```
# 每天早上6点 
0 6 * * * echo "Good morning." >> /tmp/test.txt //注意单纯echo，从屏幕上看不到任何输出，因为cron把任何输出都email到root的信箱了。

# 每两个小时 
0 */2 * * * echo "Have a break now." >> /tmp/test.txt  

# 晚上11点到早上8点之间每两个小时和早上八点 
0 23-7/2，8 * * * echo "Have a good dream" >> /tmp/test.txt

# 每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点 
0 11 4 * 1-3 command line

# 1月1日早上4点 
0 4 1 1 * command line SHELL=/bin/bash PATH=/sbin:/bin:/usr/sbin:/usr/bin MAILTO=root //如果出现错误，或者有数据输出，数据作为邮件发给这个帐号 HOME=/ 

# 每小时（第一分钟）执行/etc/cron.hourly内的脚本
01 * * * * root run-parts /etc/cron.hourly

# 每天（凌晨4：02）执行/etc/cron.daily内的脚本
02 4 * * * root run-parts /etc/cron.daily 

# 每星期（周日凌晨4：22）执行/etc/cron.weekly内的脚本
22 4 * * 0 root run-parts /etc/cron.weekly 

# 每月（1号凌晨4：42）去执行/etc/cron.monthly内的脚本 
42 4 1 * * root run-parts /etc/cron.monthly 

# 注意:  "run-parts"这个参数了，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是文件夹名。 　 

# 每天的下午4点、5点、6点的5 min、15 min、25 min、35 min、45 min、55 min时执行命令。 
5，15，25，35，45，55 16，17，18 * * * command

# 每周一，三，五的下午3：00系统进入维护状态，重新启动系统。
00 15 * *1，3，5 shutdown -r +5

# 每小时的10分，40分执行用户目录下的innd/bbslin这个指令： 
10，40 * * * * innd/bbslink 

# 每小时的1分执行用户目录下的bin/account这个指令： 
1 * * * * bin/account

# 每天早晨三点二十分执行用户目录下如下所示的两个指令（每个指令以;分隔）： 
203 * * * （/bin/rm -f expire.ls logins.bad;bin/expire$#@62;expire.1st）　
```


