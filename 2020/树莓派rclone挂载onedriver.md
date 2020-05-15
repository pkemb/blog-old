<h1 id=file>
    树莓派rclone挂载OneDrive
</h1>

rclone是一个从云端同步文件的命令行工具，常见的网盘和协议基本上都支持，更加详细的介绍可以查看[官网](https://rclone.org/)。这还是一个开源项目，[github](https://github.com/rclone/rclone)。

<h1 id=toc>
    目录
</h1>

[环境准备](#prepare)

[安装rclone](#install_rclone)

[rclone配置OneDriver账户](#rclone配置OneDriver账户)

[挂载](#mount)

<h1 id=prepare>环境准备</h1>

* 一台可以正常运行的树莓派
* [宝塔面板](http://www.bt.cn/)
  * 新增网站和配置反向代理。如果可以手动配置，也可以不安装。
* OneDriver账号，OneDriver个人账户可能无法正常登录，推荐OneDriver for business。

<h1 id=install_rclone>安装rclone</h1>

参考文档：[https://rclone.org/downloads/](https://rclone.org/downloads/)

使用以下命令一键安装。如果因为网速原因导致下载失败，可以参考[安装脚本](https://rclone.org/install.sh)的流程，手动下载并安装。

```shell
curl https://rclone.org/install.sh | sudo bash
```

安装完成之后，可以执行以下测试命令，测试是否安装成功。如果提示`rclone: command not found`，则表示安装失败。否则安装成功。

```shell
rclone --help
```

<h1 id=rclone_config_onedriver>rclone配置OneDriver账户</h1>

参考文档：[https://rclone.org/onedrive/](https://rclone.org/onedrive/)

<h2 id=get_client_id>获取客户ID和密码</h2>

以下是官方给出的获取步骤，注意保存好客户端ID和密码，关闭页面后将无法再次查看。第5步给API设置的权限，不能少，否则会影响后续对文件的操作。

> 1. Open [https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade), then `click New registration`.
> 2. Enter a name for your app, choose account type `Any Azure AD directory - Multitenant`, select `Web` in `Redirect URI` Enter `http://localhost:53682/` and click Register. Copy and keep the `Application (client) ID` under the app name for later use.
> 3. Under `manage` select `Certificates & secrets`, click `New client secret`. Copy and keep that secret for later use.
> 4. Under `manage` select `API permissions`, click `Add a permission` and select `Microsoft Graph` then select `delegated permissions`.
> 5. Search and select the follwing permssions: `Files.Read`, `Files.ReadWrite`, `Files.Read.All`, `Files.ReadWrite.All`, `offline_access`, `User.Read`. Once selected click `Add permissions` at the bottom.

<h2 id=config_rclone>配置rclone</h2>

在配置rclone的过程中，需要弹出浏览器进行认证。由于树莓派没有图形界面，所以需要曲线救国，有两种思路：
1. 先在Windows下安装rclone，并配置好OneDrive，获取token。最后在树莓派上使用token设置OneDrive，具体步骤可以参考这篇[博客](https://www.xiaoz.me/archives/10397)。
2. 利用反向代理，实现间接访问。

rclone配置步骤：略。

输入OneDrive的客户端ID和密码之后，会提示下图所示的内容，现在需要访问`http://127.0.0.1:53683`，进行认证。这里使用反向代理的方法，实现间接访问。

![](https://gitee.com/pkemb/pic/raw/master/image/20200515231022.png)

反向代理：
1. 进入宝塔面板的`安全`界面，打开`53683`端口。
2. 进入宝塔面板的`网站`界面，添加一个站点。假设域名是`rclone.pi3b.local`。
3. 点击新增网站的`设置`按钮，进入`反向代理`，将网站代理到`http://127.0.0.1:53683`。
![](https://gitee.com/pkemb/pic/raw/master/image/20200515231732.png)
1. 添加域名的DNS解析。
2. 在Windows访问`rclone.pi3b.local/`，点击授权，然后会自动跳转到`localhost:53683`，这时只需手动将`localhost:53863`更改为`rclone.pi3b.local`即可。这时即可完成认证。
![](https://gitee.com/pkemb/pic/raw/master/image/20200515232012.png)

<h1 id=mount>挂载</h1>

```shell
#安装fuse
yum -y install fuse
#创建挂载目录
mkdir -p /home/onedrive
#挂载
rclone mount remote:path/to/files /home/onedrive
#如果需要后台保持运行，使用下面的命令
nohup rclone mount remote:path/to/files /home/onedrive &
# 检查挂载是否成功
df -h
```

`remote`是设置OneDrive时配置的名字。

