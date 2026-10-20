# Step 1：基础搭建

> ⚠️ **免责声明**
> 本教程仅用于个人学习交流，严禁商用；因使用本教程产生违法行为的，责任与作者无关。
> 本人仅整理改进，部分案例教学来自网络，如有侵权请联系删除。

## 准备工作

注册并登录独角鲸云，完成充值后进入仪表盘界面。本教程以独角鲸云的实例为示范，所有用到的命令都汇总在文末。

## 第一步：新建实例

点击左侧「新建实例」，选择需要的地区、套餐与配置：

1. 系统选择 **Debian**
2. 密码建议自定义

> 💡 这类机器是 NAT 小鸡，常用端口不对外开放，只能通过**端口转发**访问。其中，**外部端口**是我们连接时使用的端口。

![新建实例](https://github.com/user-attachments/assets/60b41b01-eb2d-4413-9ed7-2c2a9df79470)

设置好端口转发后，就可以进入服务器开始搭建了。连接工具有很多，本教程使用官方提供的 Web Shell 演示。至此，第一步完成。

![进入 Web Shell](https://github.com/user-attachments/assets/fb95b58a-3668-4769-9808-38866663a89c)

## 第二步：搭建 3x-ui

小鸡自带的系统是 Debian 13.6，已经启用 BBR 加速。

先进行系统环境清理与依赖安装，然后安装 3x-ui **v2.9.4**（本教程是单节点场景，使用此版本）。

![安装 3x-ui](https://github.com/user-attachments/assets/a8d59fe0-6c17-4da1-b9d9-564ff352b508)

安装完成后，查看面板的访问路径和地址：

![查看面板地址](https://github.com/user-attachments/assets/4f0f70f8-4768-4749-9342-ec3d89786c90)

把面板地址复制出来，将其中的端口改为第一步设置的**外部端口**，得到的就是真实可访问的面板地址：

![修改面板端口](https://github.com/user-attachments/assets/3aecd3d6-3d7d-4715-8c65-d3f7de251ba9)

设置面板开机自启动，并重启面板：

![设置自启动并重启](https://github.com/user-attachments/assets/1a2d8192-fccb-43d0-b7e1-7d13c371b65a)

接着设置面板的用户名和密码：

![设置用户名和密码](https://github.com/user-attachments/assets/bd53e086-7c04-4e50-8bd4-45d2a5081d50)

设置好账号密码后，就可以退出脚本菜单了。至此，第二步完成。

![退出脚本菜单](https://github.com/user-attachments/assets/9415bcd7-b106-428a-99dd-2b1963968b4e)

## 第三步：配置 3x-ui 并添加节点

用上面得到的地址进入 3x-ui 面板。

> ⚠️ **注意：记得把地址中的端口改为外部端口，才能进入面板。**

在面板中添加我们的 VPN 节点：

![添加节点（一）](https://github.com/user-attachments/assets/68a4568c-1fea-4705-8d04-45408b606fd8)

![添加节点（二）](https://github.com/user-attachments/assets/32603453-7b96-44fb-abcd-c790a6deb6f1)

![添加节点（三）](https://github.com/user-attachments/assets/86e2ecdf-4b4a-46cf-8c3a-60160dab67cc)

节点添加完成后，把节点地址复制到 VPN 客户端即可使用。**记得将节点端口修改为之前配置的外部端口。**

## 附：本篇用到的全部命令

```bash
# 系统环境清理与依赖安装
apt update && apt install -y curl tar ca-certificates && apt clean

# 安装 3x-ui v2.9.4（安装脚本链接待补，见 3x-ui 官方仓库）
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh) v2.9.4

# 查看面板设置
x-ui settings

# 启动面板
x-ui start

# 设置开机自启动
x-ui enable

# 重启面板
x-ui restart

# 打开管理菜单
x-ui
```

---

**下一篇：** [Step 2：故障排查](step2故障排查.md)
