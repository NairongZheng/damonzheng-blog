# note

## linux操作相关

### 安装与设置

1. 安装ping等工具：`sudo apt-get install iputils-ping`
2. 安装网络工具（如ifconfig）：`sudo apt-get install net-tools`
3. 安装telnet：`apt-get install telnet`
4. 设置终端提示符样式（`~/.bashrc`里）：`PS1="\\e[1;37m[\\e[m\\e[1;32m\\u\\e[m\\e[1;33m@\\e[m\\e[1;35m\\h\\e[m:\\e[m\\$PWD\\e[m\\e[1;37m]\\e[m\\e[1;36m\\e[m$ "`

### 查看信息

1. 查看linux版本等信息：`cat /etc/os-release`
2. linux查看当前文件夹中各文件大小：`du -sh ./*`或者用`ls -lh`
3. 查案磁盘使用情况：`df -h`
4. 查看内存：`free -h`

### 用户管理

1. 新建用户：`sudo useradd -m -s /bin/bash <user_name>` && `sudo mkdir -p /home/<user_name>`
2. 删除用户（保留主目录）：`sudo userdel <user_name>`
3. 删除用户及其主目录：`sudo userdel -r <user_name>`
4. 设置用户进root无需密码：
   1. `sudo visudo`
   2. `<%sudo ALL=(ALL:ALL) ALL>`下面添加`<your_username> ALL=(ALL) NOPASSWD:ALL`
   3. `ctrl + x` 保存再回车就退出

### 文件操作

1. 文件解压
   1. zip文件解压：`unzip <filename>`
   2. tar文件解压：`tar -xvf <filename>`
   3. tar.gz文件解压：`tar -zxvf <filename>`
   4. rar文件解压：`unrar x <filename>`
   5. 7z文件解压：`7za x <filename>`
2. 文件复制
   1. 复制文件夹并排除特定子文件夹：`rsync -av --exclude="<subfolder1>" --exclude="<subfolder2>" --exclude="<subfolder3>" <source_dir> <destination_dir>`
   2. 例如：`rsync -av --exclude="*.T" --exclude="znr" --exclude=".git" ../up/bwy_v4/bwy ./`
   3. 其中，`-a`表示归档模式，表示递归复制并保持文件属性。`-v`表示详细模式，显示复制过程中的详细信息。
3. 文件传输（scp）
   1. 本地传到服务器：`scp -P 60011 <filepath_windows> root@192.168.18.13:<filepath_linux>`
   2. 服务器传到本地：`scp -P 60011 root@192.168.18.13:<filepath_linux> <filepath_windows>`
   3. 注意，因为开发机有防火墙之类的东西，所以没办法上传成功。要用这种方法传的话，需要在docker开个端口，可以直接用ssh连接docker，往docker映射到开发机的路径传就可以。所以上面的命令用的是root，而不是username
4. 文件传输（sz/rz，mobaxterm为例）
   1. 本地上传到服务器：`rz` && `ctrl + 鼠标右键` && `Send file using Z-modem` && `选择文件`
   2. 服务器下载到本地：`sz filename` && `ctrl + 鼠标右键` && `Receive file using Z-modem`
   3. 中途取消操作：`ctrl + x`按4到5次
5. 查看md5值：`md5sum <filename>`

### 网络端口操作

1. 查看本机ip：`ifconfig`
2. 查看所有端口使用情况：`netstat -tunlp`
   1. \-t (tcp) 仅显示tcp相关选项
   2. \-u (udp)仅显示udp相关选项
   3. \-n 拒绝显示别名，能显示数字的全部转化为数字
   4. \-l 仅列出在Listen(监听)的服务状态
   5. \-p 显示建立相关链接的程序名
3. 查看某端口监听状态（端口占用时候可以看）：`lsof -i:<port>`
4. 查看防火墙规则：`iptables -L`
   1. 将输出的结果询问gpt是什么意思，给出以下回答
   2. 总的来说，输出显示了防火墙的配置情况，但并没有明确指出是否有针对公网 IP 地址的特定配置。要确保公网可以访问到指定端口，你需要确保在相应的链（比如 INPUT 链）中有允许公网 IP 地址访问指定端口的规则。
5. 测试能否连上某ip跟port：`telnet <ip> <port>`
6. socket连接测试，跟规则有关系，下面给的是px的测试：

{% content-ref url="socket-lian-jie-ce-shi.md" %}
[socket-lian-jie-ce-shi.md](socket-lian-jie-ce-shi.md)
{% endcontent-ref %}

### ssh操作

1. 重启ssh服务：`/etc/init.d/ssh restart`
2. 生成密钥对：
   1. 从文件生成：`ssh-keygen -t rsa -f </file/to/private_key>`
   2. 直接生成密钥对：`ssh-keygen -t rsa -C <some tag such as email>`

### 其他操作

1. 查看当前使用的shell：`echo $SHELL`或者`echo $0`
2. [重定向输出到黑洞](https://blog.csdn.net/longgeaisisi/article/details/90519690)：`/dev/null 2>&1`
3. proto编译：`cd <proto_file_path>` && `python -m grpc_tools.protoc -I./ --python_out=./ --grpc_python_out=./ ./<proto_file_name.proto>`

## windows操作相关

#### 网络端口操作

1. 查看本机ip：`ipconfig`

### 其他操作

1. 开启WSL功能：管理员模式在`PowerShell`中运行`dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`

## git相关

1. [权限/系统等问题导致的diff](https://www.jianshu.com/p/3b8ba804c47b)：

```bash
# 具体命令可以使用的时候查，可以参考下面的

# 当文件夹更改权限（以下命令）或者更改系统时候，git可能会出现很多diff
sudo chmod -R 777 file
sudo chown -R ${username}:${username} file
# 若不想管理权限导致的diff可以用下面这条命令
git config --add core.filemode false
# 系统换行导致的diff可以使用下面命令取消
git config --global core.autocrlf true
```

1. git log设置

```bash
# 在~/.bashrc中添加以下内容
alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'
alias ll='ls-al'
# 然后执行source ~/.bashrc生效
```

## docker相关

1. 启动docker服务：`service docker start`
2. 查看docker磁盘使用情况：`docker system df`
3. 清理build缓存：`docker builder prune`
4. 容器提交到镜像：`docker commit <container_name> <newimage_name:tag>`
5. 保存镜像到本地文件：`docker save -o <file_path/file_name.tar> <image_name:tag>`
6. 加载从本地保存的镜像：`docker load -i <image_name.tar>`
7. 查看正在运行的docker的日志（若有）：`docker logs <container_name> --tail 10 -f`
8. 查看镜像构建过程：`docker history <image_name:tag>`
9. 查看容器/镜像的详细信息：`docker inspect [container_name|image_id]`
10. 连接容器：
    1. `docker attach <container_name>`
    2. `docker exec -it <container_name> bash`
    3. `attach`将连接到容器的主进程，可能会影响容器的运行（如使用 Ctrl+C 可能会停止容器）。
    4. `exec`启动一个新的进程，而不是连接到已有的主进程，不会影响容器的主进程，退出新进程不会停止容器。
    5. 也就是说，假如这个容器有启动命令，一直在前台运行某个服务，attach进去之后，其实没办法操作，只能Ctrl+C停止进程，而容器一般都使用-d -rm 之类的命令启动的，这么做就会使容器直接停止并删除。而使用exec进去之后是新开了一个进程，并不会影响主进程的运行，因此很适合进入有启动命令的容器查看相关信息。（总之推荐用exec！！！）

## 环境相关

1. [python项目自动生成环境配置文件requirements.txt](https://blog.csdn.net/pearl8899/article/details/113877334)：`pipreqs .`
2. 导出conda环境配置：`conda env export > environment.yml`
3. [mobaXterm使用本地conda](https://www.cnblogs.com/AnonymousDestroyer/p/17258702.html)：在`~/.bashrc`中添加以下代码：

```bash
export PATH=/drives/d/app/anaconda/install/Scripts:$PATH
export PYTHONIOENCODING=utf-8
if [[ "${OSTYPE}" == 'cygwin' ]]; then
    set -o igncr
    export SHELLOPTS
fi
```

## python相关

1. 打包：`pyinstaller --onefile <py_file_path>`

## 问题

1. [\[linux中按上下左右键为什么变成^\[\[A^\[\[B^\[\[C^\[D](https://www.zhihu.com/question/31429658)：输入`bash`解决

