# SYSU-SyncdialAttempt
在SYSU对校园网的一些研究和对多拨的研究，全文4500字左右，望能给大家分享一些经验，另外如果各位有能够实现进一步功能的请不在issue中指点一二

# 个人NAS与宿舍网络改造纪实

## 一、NAS构建历程

### 1. 项目背景
这个想法在我脑海中萦绕已久。一个月前经过需求自我辩论后得出结论：既然总是惦记着，不如直接配一台。

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
- 基于Debian的国产NAS系统
- 遇到问题：硬盘性能导致系统卡死
- 排查发现主板SATA接口和IPMI控制可能存在问题

#### 3.2 TrueNAS体验
- 国际流行的NAS系统
- 优点：存储配置和性能优化优秀
- 缺点：UI不友好，数据迁移困难

#### 3.3 最终方案：Windows Server
- 使用Datacenter版本
- 优势：
  - 优秀的服务管理
  - 低资源占用（开机仅4G内存）
  - 原生支持exFAT/NTFS
- 版本迭代：
  - 初始安装2022版
  - 误删Admin账户后重装为2025版

### 4. 软件配置

#### 4.1 下载管理
- 尝试方案：qBittorrent
- 最终方案：百度网盘（年费会员168元）
- 额外工具：PeerBanHelper（防止PCDN吸血）

#### 4.2 Docker环境
- 通过WSL2实现
- 运行服务：
  - Immich（相册管理）
  - Emby Server（媒体管理）

#### 4.3 媒体管理
- 对比方案：
  - Plex：不支持本地多账号
  - Jellyfin：刮削能力弱
- 最终选择：Docker版Emby Server

### 5. 待解决问题
1. Immich目录映射异常
2. 负载均衡优化（目前无线网卡利用率低）

---

## 二、宿舍多拨网络构建

### 1. 项目背景
受网络论坛启发，尝试通过多拨突破校园网单设备限速。

### 2. 技术方案
核心组件：
- `minieap`：锐捷认证破解
- `syncdial`+`macvlan`：多拨实现
- `mwan3`：负载均衡

### 3. 硬件准备
1. 现有设备：刷梅林固件的斐讯K3
2. 新增设备：
   - 红米AC2100（50元）
   - 新华三Magic S2G交换机（35元）

### 4. 实施过程
#### 4.1 OpenWRT刷机
- 使用Breed引导
- 测试多个固件版本
- 最终方案：ImmortalWRT + Kenvix插件

#### 4.2 遇到的问题
1. 天花板网口未启用（网络中心确认）
2. 多拨认证冲突（始终有一个接口掉线）

### 5. 最终方案
- 使用梅林固件原生双WAN功能
- 拓扑结构：
