---
title: 'Error: Comments Not Initialized'
date: 2018-06-05 15:53:54
tags: [Hexo,主题,gitment,原创]
categories: 
  - 主题插件
---

第一次用Hexo创建博客时用的是[mellow主题](https://github.com/codefine/hexo-theme-mellow)，该主题里评论系统设置了两个：

1. valine

   ```
   valine:
     enable: false
     appid: #appid
     appkey: #appkey
     notify: true  #是否开启邮箱提醒
     verify: true  #是否开启验证码
     placeholder: give me some sugers plz... #留言板中的预留信息
     avatar: 'wavatar'  #用户头像
   ```

   

2. gitment

   ```
   gitment:
     enable: true
     lazy: true 
     owner: ##gitHub账号名 
     repo: ##gitHub账号名下创建一个gitment的项目（可以是个空项目） 
     oauth: 
     client_id: ##创建OAuth Apps后生成的Client ID
     client_secret: ##创建OAuth Apps后生成的Client Secret
     perPage: 10 
   ```

   

我用到的则是第二个，在按照该作者的[详细文档](https://github.com/codefine/hexo-theme-mellow/wiki/4.-%E5%9F%BA%E4%BA%8Egithub%E7%9A%84%E8%AF%84%E8%AE%BA%E7%B3%BB%E7%BB%9F-gitment)设置后在博客页面上始终提示Error: Comments Not Initialized，点击Login则跳转到博客主页

后来实在没办法只好问作者本人，解决办法如下：

1. 通过nginx配置把带www的URL跳转到一级域名 （这是当时我百度的，但是没解决我的问题，可能会解决别的问题，暂时加上）

2. 在头像--Settings--Developer settings--OAuth Apps中找到配置的Homepage URL 以及Authorization callback URL

    ![1528187254963](1528187254963.png)

   ，确保这两个配置跟Github的Repository Settings里的GitHub Pages---Enforce HTTPS 选中（选中则表示以https协议，不选择以http）

   ![1528187303127](1528187303127.png)

   协议一致

更改后保存直接刷新页面，然后点击Login，成功绑定，成功初始化，至此问题解决，特写下此文来记录下

