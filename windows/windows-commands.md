# Windows 桌面运维故障排查

二、Windows 桌面运维
————————————————————————————————————————————————————————————————


## 2.1 网络故障排查

使用场景：用户说上不了网 / 网页打不开 / 网络慢

  ipconfig                                    查看 IP、网关、子网掩码
  ipconfig /all                               查看详细信息（DNS、MAC、DHCP 服务器）
  ipconfig /flushdns                          清除 DNS 缓存（改 hosts 后务必执行）
  ipconfig /release && ipconfig /renew        释放并重新获取 IP

  ping 127.0.0.1                              确认网卡正常
  ping 网关IP                                  确认局域网连通
  ping 114.114.114.114                        确认外网连通
  nslookup baidu.com                          确认 DNS 解析
  tracert 114.114.114.114                     路由追踪定位断点

  netsh winsock reset                         重置网络套接字（顽固问题，需重启）
  netsh int ip reset                          重置 TCP/IP 栈（需重启）

  ★★ 标准排查顺序 ★★  网卡 → 局域网 → 外网 → DNS，逐层排查，不跳步骤


## 2.2 系统信息与激活

使用场景：查看电脑配置 / 系统版本 / 激活状态

  systeminfo | findstr /C:"OS Name"           查看系统版本
  systeminfo | findstr /C:"系统启动时间"        查看上次重启时间
  slmgr /xpr                                  查看激活过期时间
  slmgr /dlv                                  查看详细激活信息
  dxdiag                                      查看显卡型号和驱动版本
  winver                                      弹窗显示版本号
  msinfo32                                    系统信息 GUI 界面


## 2.3 设备与驱动

使用场景：打印机装不上 / USB 不识别 / 设备管理器有黄叹号

  devmgmt.msc                                 打开设备管理器（看黄叹号）
  printui                                     打开打印机管理界面
  diskmgmt.msc                                打开磁盘管理（分区格式化）

  标准排查：
  1. 设备管理器查看黄叹号 → 右键更新驱动 / 官网下载
  2. 打印机不打印 → 先看是否脱机，重装驱动
  3. USB 不识别 → 换接口 / 设备管理器卸载设备重新扫描


## 2.4 磁盘空间清理

使用场景：C 盘变红 / 装不了软件 / 电脑变慢

  cleanmgr                                    磁盘清理工具（选 C 盘清临时文件、更新缓存）
  powercfg /h off                             关闭休眠（释放 3-8GB）
  diskmgmt.msc                                查看分区情况

  ★★ 虚拟内存迁移 ★★  此电脑 → 属性 → 高级系统设置 → 性能设置 → 高级 → 虚拟内存更改
  → C 盘设"无分页文件" → D 盘设"系统管理的大小"

  查大文件：
  Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue | Sort-Object Length -Descending | Select-Object Name,@{N="MB";E={[math]::Round($_.Length/1MB,1)}} -First 20


## 2.5 域加入与组策略

使用场景：新员工电脑加域 / 加域报错 / 策略不生效

  -- 图形界面加域 --
  此电脑 → 属性 → 高级系统设置 → 计算机名 → 更改 → 域 → 输入域名 → 域管理员账号 → 重启

  gpresult /R                                 查看当前生效的组策略
  gpupdate /force                             强制刷新组策略（改了策略不生效时执行）
  whoami                                      查看当前登录账号
  netdom join 计算机名 /domain:域名 /userd:域管理员 /passwordd:*
                                              命令行加域（批量操作用）

  常见加域报错：
  找不到网络路径 → ping 域控 IP 是否通
  拒绝访问 → 确认用的是域管理员账号


## 2.6 远程协助

使用场景：帮同事远程解决问题

  mstsc                                       打开远程桌面连接
  mstsc /v:IP或计算机名                        直接连接指定电脑
  msra                                        远程协助（需对方接受邀请）

  开启被远程：设置 → 系统 → 远程桌面 → 启用远程桌面


## 2.7 软件安装与卸载

使用场景：软件报错 / 批量卸载

  wmic product get name,version               查看已装软件列表
  wmic product where "name like '%软件名%'" call uninstall   卸载指定软件
  图形界面：设置 → 应用 → 搜索 → 卸载

  常见缺失运行库：Microsoft Visual C++ Redistributable / .NET Framework / DirectX


## 2.8 蓝屏排查

使用场景：用户电脑蓝屏 / 频繁重启

  完整排查流程：
  1. 记录蓝屏错误代码（如 MEMORY_MANAGEMENT）
  2. 搜索"Windows + 错误代码"获取解决方案
  3. eventvwr.msc → Windows 日志 → 系统 → 筛选事件 ID 1001（BugCheck）
  4. 询问用户蓝屏前做了什么操作

  进入安全模式：
  设置 → 更新和安全 → 恢复 → 高级启动 → 疑难解答 → 启动设置 → 重启

  命令行修复：
  sfc /scannow                                扫描并修复系统文件
  DISM /Online /Cleanup-Image /RestoreHealth  修复系统映像

  常见蓝屏：
  MEMORY_MANAGEMENT           → 内存条故障（插拔或更换）
  VIDEO_TDR_FAILURE           → 显卡驱动问题（更新驱动）
  CRITICAL_PROCESS_DIED       → 系统文件损坏（sfc /scannow）
  SYSTEM_SERVICE_EXCEPTION    → 驱动不兼容（安全模式卸载最近驱动）


————————————————————————————————————————————————————————————————
