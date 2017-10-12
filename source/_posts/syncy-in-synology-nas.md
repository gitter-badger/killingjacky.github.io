---
title: SyncY in Synology NAS | 在群晖中使用SyncY同步百度云
tags:
  - tech
  - SyncY
  - NAS
  - docker
  - 百度云
  - 群晖
date: 2016-11-25 21:12:38
---

## 1 安装Docker套件

套件中心安装，略。

## 2 下载syncy镜像

找开Docker套件，注册表中搜索syncy，得到结果 chiunownow/syncy ，双击下载。如果报错，请转入命令行下载，ssh登入后运行

`docker pull chiunownow/syncy`

网速慢的得等待一会儿了（其实是很久很久很久。。。）

什么？不知道怎么登入ssh？请看下一步



## 3 使能ssh并登入

控制面板 -> 终端机和SNMP -> 勾选启动SSH功能

使用自己熟悉的终端登入ssh（比如windows下的putty，mac/linux下的terminal，太基础不会的自己度娘谷狗吧）

`ssh root@your-nas-address`

密码为初建管理员密码。



##  4 Syncy配置

以下都是在ssh中。

`docker run --name sss -it chiunownow/syncy /bin/bash`

现在进入了容器命令行，运行

`/usr/bin/syncy.py`

会得到这样的打印

```text
Device binding Guide:
     1. Open web browser to visit:"https://openapi.baidu.com/device" and input user code to binding your baidu account.

     2. User code: xxxxxx
     (User code valid for 30 minutes.)

     3. After granting access to the application, come back here and press [Enter] to continue.
```

一通复制粘贴点击下一步。。。。。最后ctrl+C结束。

这时完成的事情是拿到了百度云的token，同时写入了配置文件/etc/config/syncy

我们要把这个配置文件搞出来

`cat /etc/config/syncy`

复制。然后退出docker

`exit`

在DSM系统中找一个位置存放配置文件，

`mkdir -p /volume1/docker/syncy`

`cd /volume1/docker/syncy`

`cat > syncy-photo`

粘贴，ctrl+d保存。

这个默认的配置会把文件同步到百度云的“我的应用数据/SyncY/Syncy-docker目录下，如果想同步到另外的目录，或SyncY多开，可以找到`option remotepath '/Syncy-docker'`修改成想要的目录名。

最后删除临时容器

`docker rm sss`

告别命令行回到DSM的图形界面。



## 5 创建Syncy容器

打开Docker套件，在”映像“中选择”chiunownow/syncy“，"启动"，"通过向导启动"

容器名称

![image](https://cloud.githubusercontent.com/assets/5130185/20420772/cdf8802e-ad9a-11e6-833a-ed58e86529f7.png)

![image](https://cloud.githubusercontent.com/assets/5130185/20420815/0b45df8a-ad9b-11e6-8927-034c8e9207c6.png)



下一步，高级设置，卷，添加文件


![image](https://cloud.githubusercontent.com/assets/5130185/20420852/3dddfed2-ad9b-11e6-8c79-397335d6bb78.png)



添加文件夹


![image](https://cloud.githubusercontent.com/assets/5130185/20420885/6e9a9756-ad9b-11e6-9f0b-b1d629b22724.png)



注意红框内的装载路径**必须**这样填

FIX: /etc/config/syncy 只读属性去掉

"环境"，执行命令，sh /usr/bin/syncyd.sh


![image](https://cloud.githubusercontent.com/assets/5130185/20420940/cc71b7b0-ad9b-11e6-8a98-b64234eeaa04.png)



最后是这样子的


![image](https://cloud.githubusercontent.com/assets/5130185/20420553/83f50e8a-ad99-11e6-9fd1-6239d201a294.png)



启动容器


![image](https://cloud.githubusercontent.com/assets/5130185/20420975/12078746-ad9c-11e6-805c-3f734fff22db.png)



成功运行


![image](https://cloud.githubusercontent.com/assets/5130185/20421007/383c7cbe-ad9c-11e6-97d7-4574f31deb55.png)



！！注意！！
默认的配置是1小时同步一次，这种慢扫适合同步照片等不紧急备份任务，或文件数量巨大的目录(云与本地目录的比对成本很高)，想要立即同步，可重启容器。

可自行修改配置文件中的syncinterval提高同步扫描频率，比如从百度云推送文件回NAS，可设1分钟或更小，但切记文件数不要太多，这种快扫会消耗较多的CPU资源。

另外，syncy原作者针对配置项写了详尽文档，http://www.syncy.cn/index.php/syncyconfighelp/ ， 懂的自己折腾吧



## 6 多开/同步多个目录

重复4，5步 😎



## 配置项解释

```
#  synctype:[0-4]
#    [0,upload]:只检查本地文件并上传修改过的文件，忽略远端的所有修改或删除，远端删除的也不再上传
#    [1,upload+]:远端是本地的完全镜像，忽略远端的修改，远端删除的文件在下一次同步时将上传，远端新增的文件如果本地不存在，将不做任何变化
#    [2,download]:只检查远端文件是否修改，如有修改下载到本地，忽略本地的修改；如本地文件被删除，将不再下载
#    [3,download+]:检查远端和本地文件，如远端有修改，下载到本地，忽略本地的修改；如本地有文件被删除，将重新下载
#    [4,sync]:同时检查远端和本地文件，如只有远端被修改，则下载到本地；如只有本地修改，则上传到远端；如本地和远端都被修改，则以冲突设置方式为准。
#    0-3模式下，目的端自主新增的文件不会被删除
#   4模式下，当远端目录更改后，请删除本地同步根目录下的.syncy.info.db文件，否则在下次同步时将会删除本地的所有文件（系统会认为远程文件不需要被用户删除，也会删除本地的相应文件）

syncyerrlog=”
#  错误日志文件（包含路径名），为空时将输出至错误输出（默认屏幕）
# 设置值必须是指向文件，文件可以不存在（不存在时程序自动创建），父目录必须存在，不能指向已存在的目录
# 例：/mnt/sda1/log/syncyerr.log

syncylog=”
#  运行日志文件（包含路径名），为空时将输出至标准输出（默认屏幕）
# 设置值必须是指向文件，文件可以不存在（不存在时程序自动创建），父目录必须存在，不能指向已存在的目录
# 例：/mnt/sda1/log/syncy.log

blocksize=’10’
# 分片上传块大小
#  默认值为 10 (10M)
#  单位 M，此大小决定了能上传的最大文件大小（文件最大大小 = blocksize * 1024）
#  分片大小必须大于等于1（1M）

ondup=’rename’
# 重名处理方式
#  默认值为 ‘rename’
#  [rename or overwrite]
#  存在重名文件时是覆盖同名文件，还是重命名文件
#  当同步模式为0，重命名新文件，命名规则为“文件名_日期.后缀”
#  同步模式为1和2时，将重命名旧文件，命名规则为“文件名_old_日期.后缀”
#  同步模式为3时，则ondup只能为overwrite，设置成rename将不生效

datacache=’on’
# 是否开启缓存
# 默认值为 ‘on’
# 同步信息数据缓存，启用有助于提高同步速度
# 请根据你路由内存的大小来决定是否开启

excludefiles=’*/Thumbs.db’
# 排除文件或文件夹，将会同时应用于本地和远端，请合理设置此值，过多的排除选项将会降低系统的处理速度
# 有多个排除项时用分号(;)隔开
# 例：’*/Thumbs.db;*/excludefilename.*’
# 默认排除以“.tmp.syy”结尾的文件，此类型文件用于记录分片上传或断点下载信息，上传或下载完成后将自动删除，如原文件被手动修改，建议同时删除此文件
# 只支持通配符*? (*代表零个或更多个任意字符，?代表零个或一个字符)

listnumber=’100′
# 每次检查获取远程的文件数
# 默认值为 100
# 同步时每次获取的远端文件列表数量，数量过大时返回的字符串长度很大，将占用更多的内存
# 路径长度较长时也应适当缩小此值

retrytimes=’3′
#失败重试次数（发生错误时的重试次数）
# 默认值 3 次

retrydelay=’3′
# 重试延时时间（秒）
# 默认值为 3 秒

maxsendspeed=’0′
# 最大上传速度（字节/秒）
# 默认值为 0(不限速)

maxrecvspeed=’0′
# 最大下载速度（字节/秒）
# 默认值为 0(不限速)

syncperiod=’0-24′
# 运行时间段
# 默认值为 ‘0-24′
# 运行时间段（小时）
# 判断规则为[0,24)即包含设定的开始时间截止于设定的结束时间
# 如想从零点至6点之间才允许运行，应设置为’0-6’，如24小时都运行，则设置为’0-24’
# 如果当前时间不在设定范围内，将每5分钟检查一次，如果设为空，则只运行一次后退出

syncinterval=’3600′
# 同步间隔时间
# 默认值为 3600(1小时)
# 每次同步完成之后与下一次开始同步的间隔时间
# 单位：秒
```
