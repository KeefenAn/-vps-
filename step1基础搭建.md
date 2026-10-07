# -vps-

        

通过云服务器租赁 自建vps机场教程 
    ！！！仅用于个人学习交流使用 严禁商用 产生违法行为责任与作者无关
    本人仅整理改进 部分案例教学来自网络 侵权必删！！！
    ！！！！所有用到的代码会放在文末！！！！
    本贴以独角鲸云的实例为示范
    注册登录充值后可进入仪表盘界面
    点击左侧的新建实例 选择需要的地区以及选择套餐与配置
    

    1，这里我们选择Debian系统
    2，建议密码自定义
    因为是NAT小鸡，常用端口是不开放的，只能设置端口转发，外部端口是我们访问的端口。

    
<img width="900" height="410" alt="image" src="https://github.com/user-attachments/assets/60b41b01-eb2d-4413-9ed7-2c2a9df79470" />

  设置好端口转发，就可以进入NAT小鸡服务器搭建了，可以使用shell客户端，这里以官方提供的Web Shell来演示，第一步完成。

  
<img width="670" height="923" alt="image" src="https://github.com/user-attachments/assets/fb95b58a-3668-4769-9808-38866663a89c" />
搭建3xui


小鸡自动的系统是debian 13.6，已经启用BBR加速。


我们先进行系统环境清理与依赖安装，



然后安装V2.9.4版本，因为我们这里是单节点VPN。


<img width="900" height="766" alt="image" src="https://github.com/user-attachments/assets/a8d59fe0-6c17-4da1-b9d9-564ff352b508" />
   
    
    安装好后进入设置查看面板路径和地址。
<img width="900" height="486" alt="image" src="https://github.com/user-attachments/assets/4f0f70f8-4768-4749-9342-ec3d89786c90" />
    
    
    将面板地址复制出来，端口修改外部端口，就是真实可访问的面板地址了。
<img width="900" height="483" alt="image" src="https://github.com/user-attachments/assets/3aecd3d6-3d7d-4715-8c65-d3f7de251ba9" />

  
    
    设置面板开机自启动后，重启面板，接着设置面板的用户名和密码。
<img width="708" height="1190" alt="image" src="https://github.com/user-attachments/assets/1a2d8192-fccb-43d0-b7e1-7d13c371b65a" />

<img width="900" height="451" alt="image" src="https://github.com/user-attachments/assets/bd53e086-7c04-4e50-8bd4-45d2a5081d50" />
       
        
        设置好账号密码就可以退出脚本菜单了，第二步完成。

<img width="662" height="1181" alt="image" src="https://github.com/user-attachments/assets/9415bcd7-b106-428a-99dd-2b1963968b4e" />

设置3xui。
根据上面获得的url进入3xui
！！！记得改端口才能进！！！
添加我们的VPN节点


<img width="900" height="341" alt="image" src="https://github.com/user-attachments/assets/68a4568c-1fea-4705-8d04-45408b606fd8" />
<img width="786" height="1669" alt="image" src="https://github.com/user-attachments/assets/32603453-7b96-44fb-abcd-c790a6deb6f1" />
<img width="794" height="1181" alt="image" src="https://github.com/user-attachments/assets/86e2ecdf-4b4a-46cf-8c3a-60160dab67cc" />



添加好节点后，就复制节点地址到我们的VPN客户端使用，记得修改端口为之前配置的外部端口。

需要的代码如下；

apt update && apt install -y curl tar ca-certificates && apt clean

bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh) v2.9.4

x-ui settings

x-ui start

x-ui enable

x-ui restart

x-ui




