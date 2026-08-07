# Git 常用命令

运维里的版本管理，记这套就够用。

## 日常四连（每天用）

  git status                    看现在改了什么
  git add -A                    把所有改动放进暂存区
  git commit -m "改了什么"       存一个版本（引号里写人话说明）
  git push                      上传到 GitHub 云端

## 开工第一句 / 换电脑

  git pull                      拉取云端最新版本

## 查看与回退

  git log --oneline             看历史版本列表
  git diff                      看这次改了哪些内容（还没 add 的）
  git diff --cached             看已 add 还没 commit 的改动
  git checkout -- 文件名         放弃某个文件的改动（回到上次 commit）
  git reset --hard HEAD         放弃所有未提交改动（慎用！）

## 第一次在新电脑用

  git config --global user.name "你的名字"
  git config --global user.email "你的邮箱"

## 运维实战场景

1. 改服务器配置（如 Nginx）前先 commit 存档，改坏了一行回退
2. 部署脚本、监控脚本用 git 管版本，git diff 看每次改了什么
3. 笔记/文档仓库持续 commit，GitHub 就是你的作品集
4. 团队协作：先 pull 再改，改完 push，冲突了 git 会标出来

## 面试话术

「我用 Git 管理配置和脚本，改配置前先 commit 存档，出问题直接回退，
平时用 git diff 对比改动，笔记仓库也放在 GitHub 上持续更新。」
