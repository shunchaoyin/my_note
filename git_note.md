
# Git 的三种状态

## 工作区、暂存区（add）、Git 仓库（commit）
```bash
git init
git add .
git commit -m "message"
git commit -am "message"
git status
git log
git diff HEAD -- ReadMe.md
```

## 移除暂存区文件
```bash
git restore --staged ReadMe.md
# 或者
git reset HEAD ReadMe.md  # 取消上一次操作
```

## 版本回退
```bash
git log --pretty=oneline
git reset --hard HEAD^    # 回退一个版本
git reset --hard HEAD^^   # 回退两个版本
git reset --hard HEAD~n   # 回退到 n 个版本前
git reset --hard commitID # 回退到指定的 commit ID
git reflog                # 查看包含 reset 的 log 信息
```

## 文件删除
```bash
git ls-files
# 方法 1: 在工作区删除文件，然后 git add
# 方法 2:
git rm ReadMe.md
```

## SSH 绑定 GitHub
SSH 配置步骤请查阅 Google。

## 分支操作
开发时应保持主干可随时发布，使用分支（branch）进行功能开发，功能验证通过后才合入主干。
```bash
git branch                # 查看本地分支
git branch -a             # 查看所有分支，包括远端
git checkout branch       # 切换到指定分支
git checkout -b new_branch   # 创建新分支并切换
git branch -d branchname     # 删除分支
git merge branch             # 合并分支（不要直接 merge 主干，要在主干上 merge 其他分支）
git branch -m oldbranch newbranch  # 更改分支名称（如分支名已存在需使用 -M）
```

## 远端分支操作
```bash
git push origin branch_name         # 推送本地分支到远端
git push origin :remote_branch_name # 删除远端分支
git checkout -b local_branch_name remote_branch_name  # 切换到远端分支并建立本地分支
```

## 冲突合并
合并流程：切换到主分支，合并开发分支，若有冲突，解决冲突后继续。
```bash
git checkout master
git merge dev
# 修改文件，解决冲突
git pull       # 每次 push 前最好执行 git pull
git commit -m "message"
git push
git log --graph --pretty=oneline
```

## 标签管理
```bash
git tag tag_name                # 新建标签，默认为 HEAD
git tag tag_name commitId       # 在指定提交位置打 tag
git tag -a tag_name -m "message"  # 新建标签并添加描述
git tag                        # 查看所有标签
git tag -d tag_name            # 删除标签
git push origin tag_name       # 推送指定标签到远端
git push origin --tags         # 推送所有标签
git push origin :refs/tags/tag_name  # 删除远端标签

```

# 在 Windows 上配置 GitHub 的 SSH 密钥

## 1. 检查现有 SSH 密钥
首先，检查是否已有 SSH 密钥：
```bash
ls -al ~/.ssh
```
默认情况下，公钥文件名为 `id_rsa.pub` 或 `id_ed25519.pub`。

---

## 2. 生成新的 SSH 密钥（如无）
如果没有密钥，生成一个新的：
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
如果系统不支持 `ed25519`，使用 `rsa`：
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
按提示保存密钥文件并设置密码。

---

## 3. 启动 SSH 代理
确保 SSH 代理正在运行：
```bash
eval "$(ssh-agent -s)"
```

---

## 4. 添加 SSH 私钥到代理
将私钥添加到 SSH 代理：
```bash
ssh-add ~/.ssh/id_ed25519
```
如果是 `rsa` 密钥，替换为 `id_rsa`。

---

## 5. 复制 SSH 公钥
复制公钥内容：
```bash
clip < ~/.ssh/id_ed25519.pub
```
如果是 `rsa` 密钥，替换为 `id_rsa.pub`。

---

## 6. 添加公钥到 GitHub
1. 登录 GitHub，进入 **Settings** > **SSH and GPG keys**。
2. 点击 **New SSH key**，输入标题并粘贴公钥内容。
3. 点击 **Add SSH key**。

---

## 7. 测试连接
测试 SSH 连接：
```bash
ssh -T git@github.com
```
如果看到你的用户名，说明配置成功。

---

## 8. 使用 SSH 克隆仓库
现在可以使用 SSH 克隆仓库：
```bash
git clone git@github.com:username/repository.git
```
