# Linux 运维常用命令

﻿# 运维速查手册（最终定稿第3版）

> Linux服务器运维 | Windows桌面运维 | Docker容器运维 | 面试高频故障场景


————————————————————————————————————————————————————————————————
一、Linux 服务器运维
————————————————————————————————————————————————————————————————


## 1.1 系统状态查看

使用场景：接手新机器第一时间看状态 / 值班巡检 / 领导问服务器怎么样了

  uptime                                     查看运行时长和 CPU 负载（1/5/15分钟平均值）
  free -h                                    查看内存总量和可用量（关注 available 列）
  df -hT                                     查看各分区磁盘使用率和文件系统类型
  lsblk                                      查看磁盘分区结构和挂载点
  uname -a                                   查看内核版本信息
  hostnamectl                                查看主机名和操作系统版本


## 1.2 进程管理

使用场景：服务器卡慢 / CPU飙到100% / 定位异常进程

  ps aux | sort -nrk 3 | head -5                CPU 占用前 5 的进程
  ps aux | sort -nrk 4 | head -5                内存占用前 5 的进程
  ps -ef | grep 进程名                           查找指定进程是否存在
  top -c                                        实时监控进程（按 1 看每个核，按 M 内存排序）
  htop                                          增强版 top（没装就 apt install htop）
  lsof | grep deleted                           查找已删除但未释放磁盘空间的"幽灵文件"
  lsof -i :端口号                                查看谁占用了这个端口

  ★★ kill -9 PID ★★  强制杀死进程，进程来不及清理可能丢数据。正确做法：先 kill -15（优雅关闭），不行再 -9
  nohup 命令 &           让命令在后台持续运行（退出终端也不中断）


## 1.3 systemctl 服务管理

使用场景：部署新服务 / 服务挂了 / 改完配置要重启

  systemctl start   服务名                      启动服务
  systemctl stop    服务名                      停止服务
  systemctl restart 服务名                      重启服务
  systemctl reload  服务名                      重载配置（不中断连接，Nginx 推荐用 reload）
  systemctl enable  服务名                      设置开机自启
  systemctl disable 服务名                      取消开机自启
  systemctl status  服务名                      查看服务运行状态 + 最近日志
  systemctl daemon-reload                      修改 .service 文件后重载配置

  journalctl -u 服务名 -n 50                    查看服务最近 50 行日志
  journalctl -u 服务名 --since "10 min ago"      查看最近 10 分钟日志
  journalctl --disk-usage                       查看日志占用磁盘大小
  journalctl --vacuum-size=200M                 清理日志保留 200MB


## 1.4 Nginx 运维操作

使用场景：部署 Web 服务 / 修改配置 / 排查 502

  nginx -t                                      检查配置文件语法是否正确
  nginx -s reload                               重新加载配置（不中断当前连接）
  nginx -s stop                                 停止 Nginx
  systemctl reload nginx                        通过 systemctl 重载（更规范）
  systemctl restart nginx                       通过 systemctl 重启

  tail -f /var/log/nginx/access.log             实时查看访问日志
  tail -f /var/log/nginx/error.log              实时查看错误日志（502 报错先查这里）
  tail -n 200 access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
                                                统计访问量最高的前 10 个 IP

  ★★ 配置修改标准流程 ★★  改配置 → nginx -t 检查语法 → 语法正确再 systemctl reload


## 1.5 防火墙（firewalld / iptables）

使用场景：端口不通 / 限制访问来源 / 安全基线加固

  -- firewalld（新版 Ubuntu / CentOS 7+ 默认） --
  systemctl status firewalld                    查看防火墙运行状态
  firewall-cmd --list-all                       查看当前所有放行规则
  firewall-cmd --zone=public --add-port=80/tcp --permanent     放行 80 端口
  firewall-cmd --zone=public --remove-port=80/tcp --permanent  移除 80 端口
  firewall-cmd --reload                         重载配置

  仅允许内网段访问 MySQL（安全加固常用）：
  firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="3306" accept' --permanent

  -- iptables（通用） --
  iptables -L -n                                查看规则
  iptables -A INPUT -p tcp --dport 80 -j ACCEPT        放行 80 端口
  iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT  仅允许内网 SSH
  iptables -A INPUT -s 10.0.0.0/8 -j DROP              屏蔽某个网段
  iptables-save > /etc/iptables/rules.v4                保存规则（重启不丢）

  端口不通标准排查：
  1. ss -tlnp | grep 端口   2. firewall-cmd --list-all   3. 云服务器检查安全组


## 1.6 SSH 与远程传输

使用场景：连接 ECS / 上传下载文件 / 配置安全登录

  ssh root@IP                                  密码登录
  ssh -i 密钥文件 root@IP                       密钥登录
  ssh -p 端口号 root@IP                         指定端口登录

  scp 本地文件 root@IP:/远程路径                 上传文件
  scp root@IP:/远程文件 本地路径                 下载文件
  scp -r 本地目录 root@IP:/远程目录             递归上传目录

  rsync -avz 本地目录 root@IP:/远程目录          增量同步（断点续传，比 scp 稳定）
  rsync -avz --delete root@IP:/远程目录 本地目录  远端删除的文件本地也删（精确同步）

  ★★ SSH 安全加固 ★★  编辑 /etc/ssh/sshd_config：
  PermitRootLogin no           禁止 root 直接登录
  PasswordAuthentication no    禁用密码登录（只留密钥）
  Port 2222                    改默认端口（避开扫描） → 改完 systemctl restart sshd


## 1.7 网络排查

使用场景：服务器访问不了 / 外网不通 / DNS 解析异常 / 端口未监听

  ping 114.114.114.114                         测试外网连通性
  ping 网关IP                                  测试局域网连通性
  nslookup baidu.com                           测试 DNS 解析
  dig baidu.com                                详细 DNS 信息
  curl -I http://域名                          查看 HTTP 响应头
  telnet IP 端口                               测试 TCP 端口（最常用）
  nc -zv IP 端口                               测试端口（无 telnet 时代替）
  traceroute IP                                路由追踪定位断点
  ss -tlnp                                     查看本机所有监听端口
  ip addr                                      查看 IP 配置
  ip route                                     查看路由表


## 1.8 磁盘与日志

使用场景：磁盘报警 / 日志爆满 / 空间不足

  du -sh /* | sort -rh | head -10              根目录下最大的 10 个目录
  du -sh /var/log/* | sort -rh | head -5       日志目录下最大的文件
  find / -type f -size +500M -exec ls -lh {} \; 2>/dev/null  查找大于 500MB 的文件
  df -i                                        查看 inode 使用率（小文件太多也会满）
  cat /dev/null > /var/log/大文件.log           清空日志（不删文件，不需要重启）
  truncate -s 0 /var/log/大文件.log             同上，更规范


## 1.9 用户与权限

使用场景：新员工开账号 / 离职回收 / 文件权限不对

  useradd -m 用户名                            新建用户（自动创建家目录）
  passwd 用户名                                设置密码
  usermod -aG sudo 用户名                      将用户加入 sudo 组
  userdel -r 用户名                            删除用户及其家目录

  chmod 755 文件                               权限 rwxr-xr-x（目录常用）
  chmod 600 文件                               权限 rw-------（密钥文件必须）
  chmod +x 脚本.sh                             添加执行权限
  chown 用户:组 文件                            修改文件所有者
  chown -R 用户:组 目录                         递归修改目录所有者


## 1.10 定时任务

使用场景：自动备份 / 定期清理日志 / 定时巡检

  crontab -e                                   编辑定时任务
  crontab -l                                   查看已有任务
  systemctl status cron                         确认定时任务服务在运行

  格式：分 时 日 月 周  命令
  0   3  *  *  *  /root/backup.sh              每天凌晨 3 点备份
  */30 *  *  *  *  /script/check.sh            每 30 分钟执行一次
  0   0  1  *  *  /script/monthly.sh           每月 1 号执行


————————————————————————————————————————————————————————————————
