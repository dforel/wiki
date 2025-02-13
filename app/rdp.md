# 远程桌面 (RDP) 穿透指南

::: danger 安全警告
穿透远程桌面前 **必须** 阅读 [安全指南](/bestpractice/security.md)，确保您的系统已经 **安装了最新的补丁** 并且设置了 **强登录密码**  
进行内网穿透等于 **绕过所有防火墙** 将您的计算机直接暴露于公网中，您需要自行承担由此带来的风险  
为了保证安全，**强烈推荐** 打开 [访问认证](/bestpractice/frpc-auth.md) 功能并配置一个强访问密码
:::

<app-info :time="3" :difficulty="1.5" :access="[
    { proto: 'TCP', local: '3389', method: '系统自带远程桌面连接' },
    { proto: '(可选) UDP', local: '3389', method: '(用于优化连接)' },
]" />

## 视频教程 {#video}

::: tabs

@tab Windows 10

![](@source/app/_videos/rdp-win10.mp4)

@tab Windows 7

![](@source/app/_videos/rdp-win7.mp4)

:::

## 准备工作 {#preparation}

如果您还没有启用远程桌面的话，请先 [点击这里](ms-settings:remotedesktop) 打开设置，然后打开 **启用远程桌面** 开关。

为了保证您在系统启动后无需操作就能使用远程桌面连接电脑，在安装启动器时请勾选 `安装为系统服务` 项。如果启动器已经装好了，可以直接覆盖安装一次或参考 [系统服务](/launcher/service.md) 一文安装系统服务。

## 确认目标服务 {#target-service}

::: tip
如果开启隧道的电脑和要远程连接的电脑是同一台，请跳过这一节  
然后直接使用 `127.0.0.1` 作为 **本地 IP**、`3389` 作为 **本地端口** 即可
:::

::: details 如果不是同一台电脑，点击展开详细说明

用 `远程桌面连接` 在 **开启隧道的电脑上** 访问一下 **要远程连接的电脑**。

确保远程桌面可以正常连接后，参考下图找到您的目标 IP 和端口：

![](./_images/rdp-local-service.png)

在上图的例子中，我们要穿透的 **本地 IP** 是 `192.168.1.100`，**本地端口** 是 `3389`。

:::

## 创建隧道 {#create-tunnel}

选择 **隧道类型** 为 `TCP` 后直接填写刚才获取到的**本地 IP**和**本地端口** 即可。

**强烈推荐设置访问密码**，如需详细配置教程请参阅 [配置访问认证功能](/bestpractice/frpc-auth.md)。

![](./_images/rdp-create.png)

## 启动隧道 {#start-tunnel}

启动隧道，通常情况下使用红框中的域名即可正常连接，如无法连接可以尝试使用 IP 地址连接。

如果都无法正常连接，请检查配置是否有错误、frpc是否有输出错误日志。

确认配置完全正确的话，可以尝试使用有备案的域名 CNAME 到红框中的域名并连接或更换节点。

![](./_images/rdp-4.png)

## 连接优化

::: tip
启用 UDP **可能** 会提升远程桌面的连接质量，提升流畅度，只是 **可能**  
但是 **对于某些运营商，开启 UDP 反而会降低连接质量**，请根据您的测试结果决定要不要开这个额外的隧道
:::

- 只创建一个 TCP 隧道会使您在连接时被远程桌面连接程序歧视，看起来信号很差的样子：

  ![](./_images/rdp-conn-no-udp.png)

- 比如，我们想优化这个隧道的远程桌面连接质量：

  ![](./_images/rdp-tun-udp-old.png)

- 就需要建立一个 `节点` 相同、`远程端口` 相同 的 `UDP` 隧道：

  ::: tip
  如果相同远程端口的 UDP 隧道无法创建，说明这个节点上同一个端口号的 UDP 端口被其他人占用了  
  请删掉隧道，回到前面的 [创建隧道](#create-tunnel) 重新创建一个 TCP 隧道再试一次
  :::

  ![](./_images/rdp-tun-udp-new.png)

- 然后确保在启动器中启动 **两个隧道**，两个都要启动成功：

  ![](./_images/rdp-tun-udp-launcher.png)

- 此时再进行连接，RDP 程序可能就会显示信号很好了：

  ![](./_images/rdp-conn-udp.png)

  但是使用体验不一定会提升很多，请以实际测试为准。
