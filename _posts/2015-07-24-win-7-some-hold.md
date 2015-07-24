---
layout: post
title:  "Win7 的一些坑"
date:   2015-07-24 17:29:34
categories: docs
---
# Win7 的一些坑

### win7 登录系统提示 “user profile service服务未能登陆，无法加载用户配置文件”的解决方法

步骤： 

* 一，开机按F8，从安全模式启动。 
* 二，按Windows+R，键入“regedit”,回车。 
* 三，进入：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList，最后有两个文件夹，以“s-1-5...”开头的，一个有“.bak”后缀，一个没有。把 这两个文件夹的名字互换。然后退出，重启电脑，问题就解决了。 
* 附：如果还没有解决，继续以下步骤： 
* 四，在用户的那个带“.bak”后缀的文件夹中找到refcount，右键选0   (*有用)





### WIN 7下多了个TEMP用户，导致桌面、聊天记录都没了
解决方法是：把user同级的 Default 文件夹下的 NTUSER.DAT 复制到用户名下文件夹，然后重启，TEMP用户自动消失，恢复正常。
