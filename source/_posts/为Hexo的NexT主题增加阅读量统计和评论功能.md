---
title: 为 Hexo 的 NexT 主题增加阅读量统计和评论功能
date: 2018-07-08 18:17:36
categories:
- 技术笔记
tags:
---
### 使用 leancloud 统计阅读量
1. 注册 [leancloud](https://leancloud.cn/)。
2. 进入后台，在左上角的菜单中选择“创建新应用”完成应用创建。
3. 在左侧“数据”一栏点击“创建 Class”，class名称填写为“Counter”，其他按默认设置。
4. 在“设置-基本信息-应用Key”中找到 App ID 和 App Key。
5. 编辑`themes/next/_config.yml`文件，找到`leancloud_visitors`设置，修改为如下代码：
```yaml
# Show number of visitors to each article.
# You can visit https://leancloud.cn get AppID and AppKey.
leancloud_visitors:
  enable: true
  app_id: # 你的 App ID
  app_key: # 你的 App Key
  # 担心阅读量会被随意篡改，可按https://github.com/theme-next/hexo-leancloud-counter-security 安装插件，
  # 否则直接将 security 设为 false 即可。
  security: false
  betterPerformance: false
```

### 使用“畅言”为博客增加评论功能
1. 注册[畅言](http://changyan.kuaizhan.com/)。
2. 填写站点信息，需要备案号。
*博主是在阿里云买的域名并进行备案的，流程不复杂，只是备案审核需要十来天。*
3. 在“后台总览”找到 APP ID 和 APP KEY。
4. 编辑`themes/next/_config.yml`文件，找到`changyan`设置，修改为如下代码：
```yaml
# changyan
changyan:
  enable: true
  appid: # 你的 APP ID
  appkey: # 你的 APP KEY
```

### 备注
1. 以上两个功能的相关配置完成后，运行`hexo g`和`hexo s`命令在本地预览效果。
2. 预览没问题后，运行`hexo d`部署到网站。