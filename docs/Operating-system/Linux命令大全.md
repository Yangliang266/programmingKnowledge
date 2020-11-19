**链接：** https://www.processon.com/view/5f255568637689168e4356a8#map

#### 解压

*xz文件*

```
xz -d  mysql-8.0.22-linux-glibc2.12-x86_64.tar.xz 解压成tar文件
tar xvf mysql-8.0.22-linux-glibc2.12-x86_64.tar 解压
```



*gz文件*

```
tar -zxvf mysql-8.0.22-linux-glibc2.12-x86_64.tar.gz
```



#### Linux 命令操作

- 关于本机信息的命令操作
  - 查看用户名，ip和端口
    - who  am i
  - 查看所在目录
    - pwd
  - 查看本机ip地址
    - ip a | ip address / ip a
    - ip a | grep  管道查询ip
    - ip a |more 查询所有ip相关
  - 清屏
    - clear
  - ping 域名/ip
    - ping ip /域名
  - 强制停止快捷键
    - ctrl + c
  - 重启
    - reboot
- 关于目录的命令操作
  - 路径
    - 绝对路径
      - 绝对路径是指目录下的绝对位置，直接到达目标位置，通常是从盘符开始的路径
      - 例如：c:\windows\system32\host
    - 相对路径
      - 相对路径是指由这个文件所在的路径引起的跟其他文件大的路径关系
      - 例如：当前路径为c:\windows,使用相对路径表示host文件路径为  \system32\host
  - 目录级别
    - （.）当前目录
    - （..）上级目录
    - （/）根目录
    - （~）当前登录的home目录
    - （-）记录了上一步和下一步
  - 展示目录列表
    - ls [-ald] [目录名]
    - （-a）列出全部文件，包括隐藏文件
    - （-l）列出全部的信息，包括权限
    - （-d）进查看目录本身
  - 切换目录
    - cd [目标路径] [目录级别]
    - 注意：路径可以填写内容，比如需要找到的文件名称和后缀等等
    - 