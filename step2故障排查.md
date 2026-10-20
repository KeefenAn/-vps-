
基于step1中的方法将云服务器端口与3xui链接后 极大可能出现-1ms的情况 

step2基于此进行改进与排查。

以v2rayN为例

先修改第一点；

将step中添加入站时将端口设置为内部端口改为分配的端口（部分服务器需要其他要求，如使用20000以上的端口

<img width="939" height="1313" alt="ad4aae1e3768d80861f95a77788c51d1" src="https://github.com/user-attachments/assets/896e2bcd-b0a0-4cf2-97ca-ec275c2b1bea" />


其他步骤与step1相同

然后回到云服务器添加外部端口

<img width="1557" height="1209" alt="f276c0ad4f3a07e8c11cff771e75536f" src="https://github.com/user-attachments/assets/5e0b68a4-cedd-4e57-bcc2-b04e0f787b52" />



将添加的端口内部端口改为3xui分配的端口 

随后添加至v2rayN中

<img width="1686" height="381" alt="cd8bc7f87c930393bac38ba0d59e0a3c" src="https://github.com/user-attachments/assets/22da4233-9600-4021-9bd8-7b341781c94a" />

编辑新建的配置

将端口一栏改为在云服务器中加入的外部端口即可![Uploading cd8bc7f87c930393bac38ba0d59e0a3c.png…]()

