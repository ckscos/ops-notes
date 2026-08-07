# Git 常用命令

运维里的版本管理，记这套就够用。

## 日常四连（每天用）

git status                    看现在改了什么
git add -A                    把所有改动放进暂存区
git commit -m "改了什么"       存一个版本（引号里写人话说明）
git push                      上传到云端（GitHub/GitLab/Gitee）

## 开工第一句 / 换电脑

git pull                      拉取云端最新版本

## 查看改动

git log --oneline             看历史版本列表
git diff                      看这次改了哪些内容（还没 add 的）
git diff --cached             看已 add 还没 commit 的改动

## 回滚（改坏了退回去）

git restore 文件               放弃单个文件的工作区改动，一键还原
git revert 版本号              用新提交撤销某个历史版本（生产首选，历史不被抹掉）
git reset --hard HEAD\~1       退回上一个版本并丢弃当前提交（慎用！会丢改动）
git stash                     改到一半先存起来，紧急处理完再 git stash pop 拿回来

&#x20; 注：回滚全程在本地完成，不需要连 GitHub；只要 commit 过，没 push 也能回滚

## 备份（丢了能找回来）

commit 本身就是第一层备份（本地历史）；push 到云端是第二层（防硬盘坏）
不 push 也完全能用：git init 之后本地随便 commit，GitHub 只是多一份云端副本
换电脑/硬盘坏了：git clone https://github.com/用户名/仓库.git 全部历史拉回来
可以推多个远端做双备份（如 GitHub + Gitee）
运维神器 etckeeper：自动用 Git 管 /etc 配置目录，改配置自动留痕

## 托管平台（Git 是工具，平台只是云端）

GitHub       个人作品集、开源
Gitee（码云） 国内平台，访问快
GitLab       企业最常用，可部署在公司内网当私有仓库
自己服务器    git init --bare 搭裸仓库 + SSH，不依赖任何网站

## 第一次在新电脑用

git config --global user.name "你的名字"
git config --global user.email "你的邮箱"

## 运维实战场景

1. 改服务器配置（如 Nginx）前先 commit 存档，改坏了一行回退
2. 部署脚本、监控脚本用 git 管版本，git diff 看每次改了什么
3. 笔记/文档仓库持续 commit，GitHub 就是你的作品集
4. 团队协作：先 pull 再改，改完 push，冲突了 git 会标出来
5. 生产环境回滚用 revert，不用 reset，避免抹掉历史

## 大小写注意

Linux 上完全区分大小写（a.txt ≠ A.txt），Windows 默认不区分
Windows 上纯大小写改名要用 git mv 强制（git mv README.md readme.md）
跨平台传文件大小写不一致是经典运维故障

## 面试话术

「我用 Git 管理配置和脚本，改配置前先 commit 存档，出问题用 revert/restore 回滚；
日常 push 到远端就是备份，换机器 clone 就能恢复全部历史；
公司里常用 GitLab 私有部署，GitHub 放个人仓库。」
