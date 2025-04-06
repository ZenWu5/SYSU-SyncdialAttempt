# 个人NAS与宿舍网络改造纪实

## 一、NAS构建历程

### 1. 项目背景
此想法在我脑中萦绕许久，一个月前我在进行需求自我辩论时最终得到了最终结论：既然我不配也总是想，不如就直接配了得了。

### 2. 硬件配置
经过一周需求分析和调研，最终配置如下：

| 序号 | 项目       | 型号               | 价格 |
|------|------------|--------------------|------|
| 1    | CPU        | Xeon E3 1225 v6    | 65   |
| 2    | 主板       | 超微X11SSH-F       | 200  |
| 3    | 内存       | 三星2400 8G×2      | 96   |
| 4    | 系统盘     | 海力士PC711 512G   | 165  |
| 5    | 存储硬盘   | 固态×1 + 机械×2    | 410  |
| 6    | 机箱电源   | 线下自提           | 60   |
| 7    | 有线网卡   | AC1800             | 25   |
| 8    | 散热       | -                  | 22   |
| **总计** |           |                    | **1043** |

> 注：实际总花费约1100元（含后续配件）

### 3. 系统选择与配置

#### 3.1 飞牛OS尝试
起始是打算使用飞牛OS，一个新兴的国产NAS系统，基于debian，由于debian对大多硬件支持还是不错的，另外的原因是我这套配置的来源就是使用了飞牛，虽说原文中没有后续，但是我还是先尝试了这一系统。
但是安装后出现了很多问题，尤其是一个问题十分致命，我前后测试了许多原因，最后将问题锁定在了硬盘性能上：本机在装机伊始并没有安装另一个固态硬盘，而是仅安装了之前购买的一个机械硬盘（2.5寸笔记本拆机盘）进行测试，但是发现每次使用网络传输到达30m/s以上大概两分钟或者满速70m/s即时就会导致系统卡死（命令行没有反应，GUI卡死），再加上网上提到的核显支持问题，最终放弃了之一方案。（补充：虽然最新版本确实解决了这一问题，此问题是由于本次使用的主板是超微的工控机主板，其上有8个sata接口以及ipmi控制，此处进一步解释一下ipmi，这是用于远程控制机器开关机，可视化控制以及bios/系统安装升级的板载系统，相当于是在主板上的另一套性能较弱的用于控制主板的电脑）

#### 3.2 TrueNAS体验
而后并没有打算使用windows，因为windows的分盘制度以及C:/Users/User以及注册表和缓存等实在是太过于愚蠢且难用了，于是使用了另一套国际上使用率较高的系统TrueNAS，此系统的优点在于其储存配置系统以及性能优化，安装成功后我因为其UI上的指示不清晰以及最终文件迁移过于难用而放弃。（补充：这一问题实际上在飞牛上也在遭遇，由于我之前的数据大多是以exFAT或者NTFS存储在硬盘上，没有安装剩下的两块硬盘也是在通过网络进行数据迁移，但总之是迁移工作遥遥无期且无法支持无线网卡而作罢）

#### 3.3 最终方案：Windows Server
好的我们最终回到了windows的怀抱，我使用了windows server datacenter版本。这一版本可谓是我对windows所有好感的来源，服务好用优化好，没有鸡肋的动画，也没有开机就占用8g的弱智内存优化，总之在这款2017/8年的cpu上他运行的十分顺利，16g的内存开机也仅占用了4g，以后我换电脑，比如我下一台笔记本大概会是机械革命的独显本，我大概就会给他安装winserver了。
开始我安装的是winserver2022版本（win10server版），服务什么的都配置的差不多了最后我犯蠢把Administrator账户直接删除了（属于是蠢到极限了哈哈）导致系统无法工作，最终重装了winserver2025版（win11server版）。Win11上有我已经使用习惯的文件管理器选项卡功能，在win10时没有此功能开两个窗口操作就显得有些不习惯了。经测试，网络等功能也正常，在使用驱动检测软件安装了无线网卡的wlan和蓝牙驱动后这台机子的硬件总算也是解决了，后来上网购买的打算用作TrueNAS系统盘的20块钱16g的傲腾硬盘也用于加速机械硬盘的读取速度了。
安装了windows也直接解决了数据迁移的问题，毕竟windows可以直接原生支持exFAT和NFTS，因而硬盘就即插即用了。另外在这个时候我忽然想起了系统安装的事，一开始我是使用了一个名为USM（U盘魔术师）来安装系统，但最终由于种种问题没有成功，成功了也会有主页劫持，捆绑垃圾软件的问题，于是使用了原生重装法，然而又安装了没有GUI版本的系统，又得搜索命令重新挂载系统镜像安装系统，总之最终硬件到软件这一层终于是打通了，接下来就是在这一架构之下进行功能配置了。

### 4. 软件配置
既然使用了winNAS还想要功能就没法偷懒了（开始使用飞牛就是因为其上有现成的Docker、相册管理以及下载管理，但是由于系统优化问题最终作罢），搜索网上的文章后对于这几个问题我搜索以及实践得到的解决方案分别是：Immich作为相册备份管理，qbittorrent作为bit下载管理，emby作为媒体管理平台。

#### 4.1 下载管理
先讲qbittorrent，这个就比较简单了，下载后将web管理端口暴露在局域网中就可以在网页上直接管理下载服务了，但是这一功能后续是直接没有必要了。因为我后来在搜索资源时总是要使用百度网盘，最终还是开通了年费会员，总之是168块钱解决了资源的问题，而后我又发现了百度网盘有云添加磁力下载的功能，那就确实不如在百度网盘上下载后再下载来的方便了，于是这一功能就此作罢，等到以后有能力了可以整一些bt资源做一做上传做种。谈到做种，我在下载完成资源后经常能遇到跑满我上传网速的情况，一开始我还以为我这是给bt分享的环境做贡献，后来在网上搜索才发现是有跑pcdn的在吸血，最终使用了PeerBanHelper解决了这一问题（不是我一开始还在想我连Ipv4都没有他上哪下的我的东西下这么快，原来是吸血的）。
此处解释一下pcdn：pcdn是一些使用家庭宽带为企业（爱奇艺，优酷等）提供视频传输服务并从中获益的产业，由于我国对家用宽带价格很低且不限流量，这些人便想要将这些带宽卖给企业从中获益，但是运营商也不是傻子，自然就会监测ip的下载和上传数量，一般就是比较上传流量和下载流量。由于pcdn主要是以上传流量获益，因而就要刷下载流量来防止被ban，于是他们就到bt下载中吸血，甚至会吸大学镜像站的血，评价为一群自私自利没有公德心的社会害虫，真可谓是吸血鬼了。

#### 4.2 Docker环境
然后先讲一下docker吧，这是当初为了解决linux上运行环境配置困难以及编译麻烦等问题提出的解决方案，相当于是在系统上创立了一个虚拟机，在这个虚拟机上可以直接调用别人基于这个虚拟环境配好的服务从而避免系统内部冲突以及服务卸载后残留的问题，可谓是目前稳定性与便利性的终极解决方案，大大为广大程序员同胞减少了头发掉落的数量。于是剩下的两个服务（immich以及emby）我就安装在了docker环境中。但是正如上述介绍的，docker是搭建在linux系统下的，那么我这个windows该如何调用呢，这确实是一个很大的问题，但是windows在win1终于将WSL（Windows Subsystem for Linux）功能做的比较可用了，总之windows下的Docker是以wsl为基础来运行的，你也可以选择使用windows自带的Hyper-V但是我之前没学过Hyper-V再加上网上对其配置颇有微词我就还是回到了舒适区使用WSL来做配置。

#### 4.3 媒体管理
接下来来讲剩余的功能搭建，首先是immich这个软件有现成的android软件用于相册照片上传以及人工智能照片识别，虽然说它的以词搜图功能还是依托，但是起码是有现成的上传管理。由于要这些功能要下载ai模型以及配置外在图片库，这就涉及到了docker的目录映射。我们既然要使用Docker，那肯定还是要用到其自带的文件以外的文件，那么就要涉及到Docker的映射功能，相当于是将你电脑下的目录映射到虚拟机下的一个目录上。但是在这里我就遇到了问题，ai模型的文件是映射成功了，但是地理信息的汉化包和机上图片目录疑似是没有映射成功，百思不得其解，找不到问题的来源，暂且搁置，只使用基础的手机图片上传功能。
之后是媒体管理，一开始我是打算直接在win上使用emby server或jellyfin或者plex，但是最终由于plex无法进行本地多账号管理，jellyfin媒体库信息识别刮削能力差，卸载jellyfin后8096端口仍然被jellyfin奇怪的占用且emby无法使用（jellyfin使用emby作内核）还是使用了docker emby server（开始觉得映射都有问题就不想尝试），但是后来emby的目录映射又没有问题，再重新整合了一下数据之后，emby server成功布置在了局域网中。


### 5. 待解决问题
1. Immich目录映射异常
2. 负载均衡优化（目前无线网卡利用率低），尝试了使用相同跃点数，但是还是不大行，使用windows的网卡聚合也只是跑在无线网卡上，总之最后就当做无线网的接入点了。

---

## 二、宿舍多拨网络构建

### 1. 项目背景
此想法在我某日搜索server网络配置时有人发的帖子表示可以实现多拨提升带宽，想一想我们学校按照设备（分配的ip）来限速，就觉得我们学校的校园网说不定也可以实现这一功能，遂开干。

### 2. 技术方案
核心组件：
- `minieap`：锐捷认证破解
- `syncdial`+`macvlan`：多拨实现
- `mwan3`：负载均衡
首先是要摸清楚162栋的网络的结构查看官网上的有线网连接教程得到每个宿舍有一个主网口但是我看到我们宿舍上有一个锐捷的卫星AP并且其上有四个电口，于是打算先尝试是否能使用锐捷的网口直接连接。另外在楼下其实有一个网口，但是当时此网口正在被另一舍友使用，因而没有计划使用此网口。
谈到连接，此处先说明一下本校的无线以及有限网络接入方式，无线接入方式是802.11x PEAP认证连接，这个相对简单一些，只需要在连接wifi时输入账号密码即可，无需专门的客户端。但是有线连接则需要使用学校提供的锐捷认证客户端来连接，经后来测试使用的是锐捷的私有协议，于是就需要寻找替代方案，毕竟不太可能使用学校给的原生认证软件在路由器上进行多拨，于是上网寻找相关方案，得到了一些使用openwrt系统（linux的路由器特化变体）插件来实现锐捷认证加上多拨的方案，即minieap+syncdial+macvlan+mwan3，下面一段集中介绍一下这几个插件。

### 3. 硬件准备
1. 现有设备：刷梅林固件的斐讯K3
2. 新增设备：
   - 红米AC2100（50元）
   - 新华三Magic S2G交换机（35元）

### 4. 实施过程
#### 4.1 OpenWRT刷机
已有的一个路由器是之前六七十淘到的斐讯K3，刷了恩山G大的梅林固件，此前一直使用一个无电池手机作为usb无线网接入。鉴于网上有人提到从G大固件刷到别的固件有可能导致无线失效再加上这个系统底层不是我刷的，我对其不怎了解，因而就先保留了。另外就是网上有许多帖子都在使用红米ac2100作为openwrt的入门机。也有现成的已编译好所需软件包的固件（Kenvix），因而我没有从openwrt本家包来一步步安装（编译软件实在是麻烦，再加上我没有搞的太懂，虽说我跟着网上的教程在WSL上编译了minieap，但是最终还是没有用上，站内有一个项目是minieap for openwrt，可以去那直接下下来试试，不过一般还是不用麻烦，软件源中应该有）。
最终是上闲鱼上50包邮淘到了一个红米AC2100（rm2100）跟着网上的教程刷入了breed（降级，URL指令，这里还遇到了一个路由器下不下来breed文件的问题，最终使用了更改文件哈希值加上游路由架梯子的方式解决），然后就是测试不同的固件，直接使用Kenvix的kernel1+rootf0双刷后升级的方案会导致多接口支持有问题（DHCP分配和lan口信号问题），最终使用恩山上immortalwrt双刷固件升级Kenvix固件解决了这一问题。但是不管如何配置两个端口总是会有一个掉认证，表现为dns接口接通但是其他端口未联通。

#### 4.2 遇到的问题
另外要提到的一点是，当时买的网线到了之后接上天花板上的网口时却最终发现无法获取ip，尝试了两个接口无果后询问网络中心帮助台得知为了防止ip分配混乱天花板上的网关的有线接入没有打开。。。总之最后又回到了下面的网口，鉴于之前在此网口上测试多个虚拟网口可以得到多个ip，这个网口上使用交换机来分流应该是可行的，因而我又35￥淘了一个新华三的magic s2g千兆八口交换机用于分开lan口，这个是无管理的傻瓜交换机，因而不可能在这里配置多wan整合，还好在测试后交换机上的所有网口都可以分别得到ip，可行的。
其实到了这一步时我已经放弃单线多拨了，在最开始原生使用墙上网口以及后续在路由器上配置时均无法解决两个网口无法同时使用必有一个掉线的问题，于是我将openwrt的路由器放到了交换机旁边分vlan使用双线双拨但是最终还是相同的问题，两个接口认证后必有一个掉认证，排查这一步实在费力，最终还是作罢。猜测是minieap与多拨的冲突，或者是学校的检测发力了。

### 5. 最终方案
梅林路由器有一个功能叫双WAN口，即他本身就原生支持DHCP/pppoe协议双wan口负载均衡，因而我最终的方案还是回到了原本的手机上——在原本的基础上从openwrt路由引一条网线连接到K3上使用双wan。
最终的效果大概是平时学校人多用量高时带宽最高为70Mbps（下载）左右，学校人少（例如这两天清明假期的半夜3点）时甚至能达到400Mbps的带宽。
以下附一个总结的拓扑图：
![image](https://github.com/user-attachments/assets/0efa6014-65de-4129-8848-61dc901cff6d)

