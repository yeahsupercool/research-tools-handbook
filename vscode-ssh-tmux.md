# VS Code Remote SSH + tmux：在云服务器稳定跑任务（断线不掉）
## 如何在vscode上面连接远程服务器ssh？

### 1）添加主机
- `Ctrl+Shift+P` -> 输入 `Remote-SSH:Add NewSSH Host` -> 粘贴ssh命令 `
ssh<user>@<host>-i<path/to/key.pem>`-> 选择保存到 `~/.ssh/config` (Windows中为 `C:\Users\YourUserName\.ssh\config`)

### 2) 连接主机
- `Ctrl+Shift+P` -> `Remote-ssH:Connect to Host` -> 选刚添加的主机
- 第一次会自动安装 VSCode Server，按提示确认。
- Select the platform for the remote host: 选择Linux即可

### 3) 打开远程文件夹
- 找到 open Folder按钮 -> 选服务器上的项目目录（比如矩池云上的工作目录为`/mnt`)
  
### 4) 打开远程终端
- `Ctrl+[反引号]` 打开终端(此时终端已在远程服务器上)。

## 在同一远程主机上通过不同端口做多路 SSH 连接，并在 VSCode 中同时打开同一目录
### 问题
- 在云平台上，同一台服务器通过不同的 SSH 端口对外提供访问。虽然用于登录的 SSH 命令不同（端口不同），但 HostName 相同。如果在 VSCode Remote-SSH 中直接使用相同的 Host 标识（或未区分端口的同名主机），VSCode 往往会复用同一远程目标，导致无法在两个独立 VSCode 窗口中同时打开同一台机器上的同一目录。
### 原因
- VSCode Remote-SSH 以 ~/.ssh/config 中的 Host 别名（而非 HostName/IP）标识远程目标。若两个连接使用同一个 Host 别名（即使端口不同），VSCode 可能把它们视为同一远程，从而复用连接/窗口。
### 解决方案（为每个端口定义不同的 Host 别名）
- 在本机 ~/.ssh/config 中为每个端口配置独立的 Host 条目（不同别名、相同 HostName、不同 Port）
- Add NewSSH Host之后，打开config文件（右下角会出现弹窗，包含Open Config按钮，点击即可；当然也可以手动找到这个文件在电脑上的位置，然后打开）
- config文件内容类似如下代码，修改第一行Host 后面的端口别名即可。在接下来选择Connect to Host的时候，选择该端口即可。
```bash
# 端口 A
Host gpu-a
  HostName hz-4.matpool.com
  Port 26964
  User root
  IdentityFile ~/.ssh/mykey.pem

# 端口 B
Host gpu-b
  HostName hz-4.matpool.com
  Port 27001
  User root
  IdentityFile ~/.ssh/mykey.pem
```
- 这样，gpu-a 与 gpu-b 被 VSCode 视为两个独立远程目标，可以在两个 VSCode 窗口中分别连接，并同时打开同一远程路径（例如 /mnt）。

## 在 VSCode 远程（SSH）环境中使用 tmux 运行与监控任务
### 1) 连接确认（VSCode 端）
- 连接成功后，左下角应显示：SSH: <主机名>（表示当前窗口已在远程）。
- 打开云端终端：按 `Ctrl +(反引号)`。这是云服务器上的终端，可直接跑命令。
- 自检命令（确认确实在云端）:
```bash
whoami                 # 应是服务器用户名，如 ubuntu/root
hostname               # 服务器主机名
pwd                    # 应在远程路径，如 /home/ubuntu/...
echo $SSH_CONNECTION   # 有输出表示通过 SSH 连接
```

### 2) 安装与启动 tmux（云端执行）
```bash
# 一次性安装
sudo apt-get update && sudo apt-get install -y tmux
# 新建 tmux 会话
tmux new -s job  # job为会话名称，可以更改为其他名称
# 输出形如：anti-foreign: 1 windows (created Tue Nov  4 16:06:01 2025) (attached)

# 若已经创建过对话了，可以无需创建新会话。
# 首先，查看现有会话
tmux ls
# 然后重新接入会话（eg. 接入job会话）
# 首先需要先在还没有终止会话的时候，`tmux ls`查看一下会话名称，确保知道哪个是正在运行的会话，然后再打开一个新的终端New Terminal，输入`tmux attach -t job`，即可进入之前这个会话的终端界面。
tmux attach -t job  # 输出为sessions should be nested with care, unset $TMUX to force
# 检查是否入成功
echo $TMUX   # 若有输出，则当前shell已经在tmux会话里；在这个终端里启动的任何命令/脚本，即使 SSH 断开，也会继续在服务器上运行。
```


### 3) 在 tmux 中运行脚本
```bash
# 进入项目目录（若 VSCode 以此目录打开，可省略）
# 例如：
cd /mnt 
```
#### 获取 Python 解释器的绝对路径
```bash
python -c "import sys; print(sys.executable)"
# 示例输出：
/root/miniconda3/envs/myconda/bin/python
```

### 在运行代码之前，先检查一下是否已经挂到了tmux里面
- 输入`echo #TMUX`，若有输出，则表明已经在tmux中

#### 直接用解释器绝对路径运行脚本
```bash
# 例如：
# Python解释器的绝对路径+换行符+需要运行的脚本+输出日志
/root/miniconda3/envs/myconda/bin/python \
run_model_ratio_contribution.py \
| tee -a logs-contribution.txt
```
- 注意，这里的'logs.txt'可以直接换成其他名字，因为放在同一个文件夹中，不同代码运行的产出可能会交错。

### 4) 挂起 / 重连 / 查看日志 等操作方式
- 快捷键挺方便的， 想要后台运行Python代码（关电脑、断线都不会中断），输入：`m` 便可执行相关内容，比如查看正在运行进程，通过PID终止进程等。
```bash
tmux ls                    # 查看会话列表
tmux attach -t job         # 重新接入会话，回到实时输出窗口
tail -f logs.txt           # 实时查看日志
tail -n 100 -f logs.txt   # 查看最后100条日志
Ctrl + b，松开，接着按下 d 键  # detach某个会话
```
