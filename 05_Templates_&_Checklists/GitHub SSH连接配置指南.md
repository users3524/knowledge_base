# GitHub SSH 连接配置指南

> 适用场景：在境内或受限网络环境下，`git push` 使用 HTTPS 远程地址时频繁出现连接重置、超时或认证失败。推荐将仓库远程地址切换为 SSH，并用密钥完成认证。

## 1. 典型故障现象

执行 `git push` 时，终端可能出现类似报错：

```bash
git.exe push --progress -- "origin" main:main
fatal: unable to access 'https://github.com/users3524/important_doc.git/': Recv failure: Connection was reset
git 未能顺利结束 (退出码 128)
```

常见表现：

- `git pull` 可以成功，但 `git push` 失败。
- 浏览器可以打开 GitHub，但 Git 命令行推送不稳定。
- HTTPS 地址反复报 `Connection was reset`、`Failed to connect`、`RPC failed`。

原因通常是 HTTPS push 会产生较大的双向传输流量，容易被代理、防火墙或深度包检测策略中断。SSH 使用密钥认证，链路通常更稳定，也更适合长期固定使用。

## 2. 检查当前远程地址

在仓库根目录执行：

```bash
git remote -v
```

如果看到类似下面的 HTTPS 地址，说明当前还没有切换到 SSH：

```bash
origin  https://github.com/users3524/important_doc.git (fetch)
origin  https://github.com/users3524/important_doc.git (push)
```

推荐目标格式：

```bash
origin  git@github.com:users3524/important_doc.git (fetch)
origin  git@github.com:users3524/important_doc.git (push)
```

## 3. 生成 SSH 密钥

优先使用 ED25519 密钥：

```bash
ssh-keygen -t ed25519 -C "users3524"
```

提示说明：

- 保存路径保持默认即可，通常是 `~/.ssh/id_ed25519`。
- 是否设置 passphrase 按个人安全需求决定；个人电脑可为空，公共设备建议设置。
- 私钥文件是 `id_ed25519`，必须保留在本机，不能上传或泄露。
- 公钥文件是 `id_ed25519.pub`，需要添加到 GitHub。

## 4. 复制公钥

Git Bash：

```bash
cat ~/.ssh/id_ed25519.pub
```

PowerShell：

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

复制完整输出内容。它通常以 `ssh-ed25519` 开头，以注释信息结尾，例如：

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... users3524
```

## 5. 添加公钥到 GitHub

操作路径：

1. 登录 GitHub。
2. 点击右上角头像，进入 `Settings`。
3. 打开 `SSH and GPG keys`。
4. 点击 `New SSH key`。
5. `Title` 填写易识别的设备名，例如 `Workstation-PC`。
6. `Key` 粘贴完整公钥内容。
7. 点击 `Add SSH key` 保存。

## 6. 测试 SSH 连通性

执行：

```bash
ssh -T git@github.com
```

第一次连接时可能出现主机指纹确认：

```text
The authenticity of host 'github.com (...)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

确认连接 GitHub 官方域名后，输入：

```text
yes
```

成功结果：

```text
Hi users3524! You've successfully authenticated, but GitHub does not provide shell access.
```

如果出现下面的结果，说明 GitHub 还没有识别到你的公钥：

```text
git@github.com: Permission denied (publickey).
```

优先检查：

- 公钥是否已添加到 GitHub。
- 是否复制了 `.pub` 公钥，而不是私钥。
- 本机是否使用了正确的私钥文件。
- GitHub 账号是否有目标仓库的写入权限。

## 7. 切换仓库远程地址为 SSH

在项目根目录执行：

```bash
git remote set-url origin git@github.com:users3524/important_doc.git
git remote -v
```

确认输出已经变为：

```bash
origin  git@github.com:users3524/important_doc.git (fetch)
origin  git@github.com:users3524/important_doc.git (push)
```

首次推送并建立上游分支：

```bash
git push -u origin main
```

后续提交只需要：

```bash
git push
```

## 8. 可选：固定 SSH 配置

如果有多个 GitHub 账号或多个密钥，建议创建 SSH 配置文件。

文件路径：

```text
~/.ssh/config
```

示例配置：

```sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes
```

Windows PowerShell 可用以下命令确认文件位置：

```powershell
notepad $env:USERPROFILE\.ssh\config
```

## 9. 备用方案：SSH 走 443 端口

如果网络封锁了 SSH 默认端口 `22`，可以改用 GitHub 提供的 `ssh.github.com:443`。

编辑 `~/.ssh/config`：

```sshconfig
Host github.com
  HostName ssh.github.com
  User git
  Port 443
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes
```

然后重新测试：

```bash
ssh -T git@github.com
```

## 10. 备用方案：HTTPS 代理

如果必须继续使用 HTTPS，并且本机代理端口为 `7890`，可以临时配置 Git 代理：

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

检查当前代理：

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

网络恢复正常后建议取消全局代理：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 11. 快速排查清单

- [ ] `git remote -v` 已切换为 `git@github.com:用户名/仓库名.git`。
- [ ] `ssh -T git@github.com` 能看到 `successfully authenticated`。
- [ ] GitHub 已添加 `id_ed25519.pub` 公钥。
- [ ] 本机私钥 `id_ed25519` 没有被删除、移动或泄露。
- [ ] 当前 GitHub 账号拥有仓库写入权限。
- [ ] 如果 `22` 端口不可用，已尝试 `ssh.github.com:443`。

## 12. 常用命令汇总

```bash
# 查看远程地址
git remote -v

# 生成 ED25519 密钥
ssh-keygen -t ed25519 -C "users3524"

# 查看公钥
cat ~/.ssh/id_ed25519.pub

# 测试 GitHub SSH
ssh -T git@github.com

# 切换仓库为 SSH 地址
git remote set-url origin git@github.com:users3524/important_doc.git

# 首次推送并设置上游分支
git push -u origin main
```
