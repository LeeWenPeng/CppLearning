## 配置环境

+   VMware Workstation Pro
    +   VMware Tools

+   XShell、Xftp
+   Visual Studio code
    +   Chinese (Simplified) (简体中文) Language 
    +   Remote Development
    +   C/C++
    +   C/C++ Extension Pack
    
+   Ubuntu 18.05.1
    +   GCC


### 虚拟机配置

详情参考：[vmware](..\Study\软件 - vmware.md)

### Xshell配置

1.   安装 ssh 服务端：`sudo apt install openssh-server`
2.   安装网络包：`sudo apt install net-tools`
3.   查看虚拟机 ip：`ifconfig`
4.   打开XShell，并新建连接："主机"选项中输入获取的远程 IP，其余默认
5.   登录：输入服务端用户名和密码
5.   连接成功

远程连接不需要输密码的技巧：[](..\Study\软件 - Xshell、Xftp.md#公钥认证——使远程连接不需要输密码)

### Vscode 配置

#### 配置汉化包

1.   下载语言包扩展：Chinese (Simplified) (简体中文) Language 
2.   打开命令面板：`Ctrl + Shift + P`
3.   搜索命令：键入 "display" 以筛选并显示 "Configure Display Language" 命令
4.   使用命令设置：选择"中文-简体"
5.   重启

#### 配置远程

1.   下载远程扩展：Remote Development

2.   进入远程资源管理器，并选择选择 SSH

3.   生成配置文件：点击 SSH 栏目右侧齿轮，选择`C:\User\用户名\.ssh\config`

4.   修改配置文件： 

     ```config
     Host 远程主机名
     	HostName 远程IP地址
     	User 远程用户名
     ```

5.   开始连接：点击 SSH 栏目下远程主机名右侧箭头

6.   选择系统：Linux

7.   是否继续：继续

8.   密码：输入远程主机密码

9.   连接完成：回到资源管理器，进行后续工作

注：如果进行了公钥认证，则不再需要输入密码



