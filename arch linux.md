
# Arch Linux

## 一 shell指令

1. 移动文件或目录

   ```shell
   mv <源文件或目录> <目标文件或目录>
   # 第二个参数：
   1. 文件。会将源文件改名为目标文件
   2. 目录。会将源文件移动到目标目录中
   ```

### 2 网络

可视化

```shell
nmtui
```

### 3 系统

1. 查看系统显卡

   ```shell
   lspci | grep -i nvidia
   ```

## 环境

### 桌面环境-sway

手册<https://wiki.archlinux.org/title/Sway>

#### 安装

```shell
sudo pacman -S sway swaylock swayidle swaybg
```

#### 配置

配置文件在 `~/.config/sway/config`

### 终端-kitty

官网 <https://sw.kovidgoyal.net/kitty/>

### vim

使用`vim <文件>`

1. `/`

   在normal modal下键入 `/`进入搜索模式

   键入想检索的单词然后回车，光标会跳到第一个匹配的单词

   按下 `n` ，光标会跳到下一个匹配的单词

   按下 `N` ，光标会跳到上一个匹配的单词

默认目录
/srv/http
/srv/http/cgi-bin
### python

#### 构建虚拟环境

##### 1 venv

python3.3+之后带有模块 venv
> 使用venv创建的虚拟环境中，使用的python版本是创建该环境的python版本
>
> 如果想创建使用其他版本的虚拟环境，需要使用python-virtualenv

1 创建

```shell
python -m venv <虚拟环境名>
```

2 激活

```shell
source <虚拟环境名>/bin/activate
```

3 退出

```shell
deactivate
```

##### 2 python-virtualenv

使用这个模块前，需要先安装这个模块

```shell
pacman -S python-virtualenv
```

1 创建虚拟环境

```shell
virtualenv <-p python<2|3>> <虚拟环境名>
# 如果不带`-p python版本号`， 则会默认使用系统自带的python版本
```

2 激活虚拟环境

```shell
source <虚拟环境名>/bin/activate
```

3 退出虚拟环境

```shell
deactivate
```

### apache

arch linux apache 的网址：<https://wiki.archlinux.org/title/Apache_HTTP_Server>

#### 1 安装

```shell
sudo pacman -S apache
```

#### 2 配置

   配置文件放在`/etc/httpd/conf`中

   配置文件为`/etc/httpd/conf/httpd.conf`

   1. 改端口
   修改`Listen 80`为`Listen 127.0.0.1:80`

   2. log地址

   错误日志在

   `/var/log/httpd/error_log`

## 软件

### pacman

pacman 使用手册 <https://wiki.archlinuxcn.org/wiki/Pacman>

#### 更新系统

```shell
sudo pacman -Syu
```

#### 安装包

1 官方包

```shell
pacman -S 包名
```

2 非官方包

1. 进入AUR仓库中找到包的github找到资源

   复制资源的`git clone url`

2. 使用git clone工具下载

   ```shell
   git clone <url>
   ```

3. 打包

    使用`pacman`中的`makepkg`

    > 不允许使用root用户运行makepkg本身
    > 如果前两步使用root用户，则`makepkg`会报**没有写入权限**的错误

    ```shell
    mkpkg -si
    # -s / --syncdeps 构建包并安装所需的依赖项
    # -r / --rmdeps makepkg稍后会删除不再需要的依赖项
    # -i / --install 满足所有依赖项并且成功构建包后，在工作目录中创建一个包文件(pkgname-pkgver.pkg.tar.zst)，使用 -i 安装
    # -c / --clean 清理剩余的文件和目录，对于包升级有用处
    ```

#### 查看包

   使用 Pacman 查看本地包数据库

   ```shell
   pacman -Qq
   ```

   >通过pacman安装日志查看是否安装某一个包
   >
   >```shell
   >[sudo] cat /var/log/pacman.log |grep apache
   >```

   查看包的详细信息

   ```shell
   pacman -Qi <包名>
   ```

#### 删除包

```shell
# 1. 最常用命令 删除指定的软件包
pacman -Rsu <包名>
```

### clash

配置文件

```shell
~/.config/clash/clash.yaml
```

更新订阅

```shell
curl -o /home/<用户名>/.config/clash/config.yaml <URL>
```

启动

```shell
clash
```

### scrot

github: <https://github.com/dreamer/scrot>

使用手册: <https://www.howtoforge.com/tutorial/how-to-take-screenshots-in-linux-with-scrot/>

安装：`sudo pacman -S scrot`

> 这个工具只适用于x11，而不适用于wayland和xwayland
