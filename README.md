# Ubuntu Station 使用指南

## 为什么需要一台ubuntu服务器？
1. 另有一台 windows 机器满足基本需求。
1. 部分 python 包不支持 windows 系统，还有一些则需要在 windows 上配置模拟 linux 的环境（MinGW）才能运行，比较复杂。
1. Pytorch 和 Tensorflow 依赖的深度学习库 CUDA 在 windows 下有性能损失。
1. 多 GPU 分布式训练神经网络在 linux 上更简单稳定。
1. windows 系统的图形界面不能多人同时使用。
1. ubuntu 是相对常见的 linux 发行版系统。大多数IT公司在实际工作中都是“ linux 服务器集群运算+ windows/mac 个人笔记本远程连接”的组合配置。

## ubuntu服务器的基本配置

- 双系统：Ubuntu 18.04 + Windows 10，自动启动 Ubuntu，必要时也可通过 BIOS 切换到 Windows
- 显卡：NVIDIA GeForce GTX TITAN(12GB显存) *2,
- 内存：64GB
- 硬盘： 256GB 固态 *2（系统盘），4TB 固态 *1（高速数据盘），4TB 机械 *3（一般数据盘）
- 出于安全原因，ubuntu 服务器的固定IP（ip_address）, 用户名（username）, 密码（passwd）等请自行询问研究组同学。

## 命令行远程连接方式ssh
1. 打开命令行终端
   - windows： win+R 打开 cmd
   - mac： 终端

2. ssh连接
   - 在本地命令行终端输入 `ssh username@ip_address`，其中具体的 username 和 ip_address 见[ubuntu服务器的基本配置](#ubuntu服务器的基本配置)。
   - 其后要求输入密码，回车确认。确认前的键盘输入不会有显示反馈，这是 linux 的保护机制。
   - ssh登录后，进入的是 ubuntu 服务器的命令行终端界面，直接通过命令操作 ubuntu 服务器

3. （optional）ssh密钥连接
可以免密码登录。[(参考)](https://www.runoob.com/w3cnote/set-ssh-login-key.html)

## 磁盘与文件系统
### 硬盘概况

|windows盘符| ubuntu 文件系统标识 |ubuntu 挂载点| 类型 | 容量 | 说明 |
| :-----| :----:| :----- | :----: | :----: |-----: |
|windows系统中不显示| /dev/sdb5 |/| 固态 | 256GB |ubuntu系统盘|
| C:\ | /dev/sdc4 |/mnt/C| 固态 |256GB|windows系统盘|
| windows系统中不显示| /dev/sda1 |/mnt/SSD| 固态 |4TB|存放训练数据,读写速度快|
| H:\ | /dev/sdf2 |/mnt/H| 机械| 4TB |一般的数据盘|
| G:\ | /dev/sde2 |/mnt/G| 机械| 4TB |一般的数据盘|
| F:\ | /dev/sdd2 |/mnt/F| 机械| 4TB |一般的数据盘|

### 文件系统说明
linux 中文件系统的逻辑与 windows 有较大区别。
在 windows 中是将每块硬盘作为根，各自组织一个树状的文件系统。如：

<html>
<table style="margin-left: 0px; margin-right: auto">
<tr>
<td>

```
C:.
├─.anaconda
│  └─navigator
│      └─logs
└─.android
   ├─build-cache
   │  └─3.4.0
   └─cache
```
</td>
<td>

```
D:.
├─360zip
│  └─config
│      └─zdefaultskin
└─Adobe
   ├─Acrobat DC
   ├─Adobe Photoshop 2022
   └─Adobe Illustrator 2022
```
</td>
<td>

```
E:.
├─AcadiaPaper
│  ├─Acadia2016
│  ├─Acadia2017
|  └─Acadia2018
└─Datasets
   └─SUNRGBD_DATASET
```
</td>
</tr>
</table>
</html>

而在 linux 系统中，所有文件都处在一棵以 `/` 为根节点的树状结构中，各块硬盘可以以个人喜好挂载（mount）在树的任意位置。
在这台ubuntu服务器中，各块硬盘按照windows下的命名传统挂载在了 `/mnt` 目录下，并在 `/home/ubuntu-station` 目录下相应创建软连接（如下图中的 C -> /mnt/C，类似于windows下的快捷方式）：
```
/.
├── bin
├── etc
...
├── home
|   └── ubuntu-station
|       ├── anaconda3
|       ├── C -> /mnt/C
|       ├── F -> /mnt/F
|       ├── G -> /mnt/G
|       ├── H -> /mnt/H
|       └── SSD -> /mnt/SSD
...
├── mnt
|   ├── C
|   ├── F
|   |   ...
|   |   └── luozn
|   |       ├── Codes
|   |       ├── data_FloorPlan
|   |       ├── data_RPLAN -> /mnt/SSD/data_RPLAN/
|   |       ├── pypotrace
|   |       ├── pytorch3d
|   |       └── 故宫调研
|   ├── G
|   ├── H
|   └── SSD
...
├── usr
└── var
```

## 常用的 linux 命令
### cd（Change Directory）
- `cd /home/ubuntu-station/F/luozn`，进入目录
- `cd ../`，进入上一级目录
- `cd ~`，进入当前用户的home目录，等价于 `cd /home/ubuntu-station/`
### ls（List Files）
- `ls /home/ubuntu-station/F/luozn`，显示目录下的内容
- `ls -l /home/ubuntu-station/F/luozn` 或 `ll /home/ubuntu-station/F/luozn`，详细显示目录下的内容
- `ls ./`，显示当前目录下的内容
### mkdir（MaKe DIRectory）
- `mkdir ./test`，在当前目录下新建test目录
### rm（ReMove）
- `rm ./a.txt`，删除当前目录下的 a.txt 文件
- `rm -r ./b/`，递归删除当前目录下的 b 文件夹
### mv（MoVe）
- `mv ./a.txt ../`，将当前目录下的 a.txt 文件**移动**到上级文件夹
- `mv ./a.txt ./a2.txt`，将当前目录下的 a.txt 文件**重命名**为 a2.txt
### cp（CoPy）
- `cp ./a.txt ../`，将当前目录下的 a.txt 文件复制到上级文件夹
- `cp –r test/ newtest/`，将 test 目录下的文件复制到 newtest 目录中
### free
- `free`，查看当前内存占用情况
### df（Disk Free）
- `df -h`，查看硬盘占用情况
### top
- `top`，实时显示 CPU 占用最高的应用
### nvidia-smi
- `nvidia-smi`，查看当前的显卡占用情况
- `watch nvidia-smi`，实时显示显卡占用情况
### Ctrl + C
- 快捷键 Ctrl + C，终止当前的进程
### conda
> 安装Anaconda后可以使用[conda命令](https://blog.csdn.net/menc15/article/details/71477949)管理python环境，以避免不同包/库版本的冲突。
> 建议为每个独立工作新建一个python环境，例如我为 FloorplanGAN，Graph2Plan 和 HouseGAN 各自建了一个独立的python环境。
- `conda info -e`，查看当前已创建的python环境
- `conda create -n SAGAN python=3.8`，创建一个新python环境，名为SAGAN，python版本为3.8
- `conda activate SAGAN`，激活SAGAN环境，其后命令行显示类似 `(SAGAN) ubuntu-station@ubuntuStation:~$`，表示当前使用的python环境为SAGAN
- `conda install matplotlib`，在当前python环境下安装matplotlib包

### jupyter notebook
> jupyter notebook 是可以通过网页使用的交互式 python 语言编辑器，以服务端和显示界面分离的方式来使用。我们一般在 ubuntu 服务器上启动 jupyter 服务，然后在远程电脑上访问使用。
> 推荐在服务器的目标环境中安装 ipykernel 包，然后从 base 环境启动 jupyter notebook。这样可以在网页界面中快速切换所需的环境[（参考）](https://www.jianshu.com/p/5eed417e04ca)。
- `(base) ubuntu-station@ubuntuStation:~/F/luozn$ jupyter notebook --ip ip_address --port 9999`，在 base 环境下，从 ~/F/luozn 目录中启动 jupyter notebook 服务。其后可以在远程电脑浏览器输入 "ip_address:9999" 进行访问使用。（具体的 ip_address 见[ubuntu服务器的基本配置](#ubuntu服务器的基本配置)）
- 注意：以上方法启动的 jupyter 服务会随着 ssh 连接的断开而关闭，所以推荐用在 tmux 会话中启动 jupyter 服务，这样可以退出 ssh 连接而保持远程服务不关闭。

### tmux（Terminal MUltipleXer）
> 通过 ssh 登录 ubuntu 服务器后直接启动的前台应用都会随 ssh 连接的断开而自动关闭。[(参考)](https://www.ruanyifeng.com/blog/2019/10/tmux.html)
> 使用 tmux 会话可以避免终止。一般的流程为：
> - ssh登录ubuntu服务器；
> - 创建 tmux 会话；
> - 在 tmux 会话中 启动深度模型训练 或 在其中启动 jupyter notebook 服务；
> - 退出 ssh 连接，关闭笔记本睡大觉;
> - 第二天重新登录服务器并进入 tmux 会话 查看模型训练情况，或打开个人电脑在浏览器输入 "ip_address：port" 继续操作运行了一晚的 jupyter notebook
- `tmux ls`，列出当前正在运行的 tmux 会话
- `tmux new -s luozn`，新建一个名为 luozn 的 tumx 会话（-s 是 session 的缩写）
- 在 tmux 会话中（底下有一个绿条），按快捷键 ctrl+b 然后再按 d 可以退出当前会话，并保持会话中的进程继续运行
- `tmux attach -t luozn`，重新进入名为 luozn 的会话
- `tmux kill-session -t luozn`，结束名为 luozn 的会话，并关闭其中的所有进程。

### sudo（Super User DO）
- 谨慎使用，冠于其它命令之前，表示用管理员权限执行命令。
- `sudo apt update`，更新软件源。

## Visual Studio Code 远程连接服务器
1. 在 vscode 的插件市场（Extensions）搜索 *Remote - SSH* 并安装
2. 点击左侧栏的远程按钮（Remote Explorer）
3. 展开 SSH TARGETS ，点击其右侧的齿轮按钮，选择第一个配置文件，添加ubuntu服务器的信息并保存：（具体的 ip_address 见[ubuntu服务器的基本配置](#ubuntu服务器的基本配置)）
```
Host Ubuntu-Station
    HostName ip_address
    User ubuntu-station
```
4. 在 SSH TARGETS 的列表中选择 Ubuntu-Station，点右侧新建窗口图标打开。其后会多次要求输入密码，不胜其烦的话可以通过[配置ssh密钥](https://www.runoob.com/w3cnote/set-ssh-login-key.html)来解决。
5. Open Folder 选择自己的目录，然后就和本地使用 vscode 一样。注意左下角有 SSH: Ubuntu-Station 表示正在远程连接中。
6. VSCode 现在也可以直接运行 *.ipynb 文件，会自动创建 jupyter notebook 服务。但同样存在关闭 VSCode 后 jupyter notebook 停止运行的问题。
