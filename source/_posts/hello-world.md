---
  title: 在GitHub上创建博客
  date: 2016-07-07 13:55:23
  tags: github
---
 
  使用Hexo及jacman主题搭建博客，将本地的Markdown博客推送到github站点。
  
  ## 准备工作
  
  * 注册Github帐号
 
    已经有Github帐号跳过此步，首先进入Github进行注册，用户名、邮箱和密码之后都需要用到，自己记好。
 
  * 个人主页
  
   每个帐号只能有一个仓库来存放个人主页，而且仓库的名字必须是username/username.github.io，这是特殊的命名约定。你可以通过 http://username.github.io 来访问你的个人主页。
 
   通过 https://pages.github.com/ 向导很容易创建一个仓库，并测试成功。不过，同样的，没有博客的结构。需要注意的个人主页的网站内容是在master分支下的。
  
  
  ### 步骤
  
  
  ``` bash
  $ npm install -g hexo
  $ hexo init "myblog"
  ```
  
  更多关于写个人博客的信息: [写作指南](https://hexo.io/docs/writing.html)
  
  ### 配置hexo
  
  ``` bash
  $ vi _config.yml
  ```
  
  配置好站点相关信息，参考：

 ``` java
 # Hexo Configuration
 ## Docs: https://hexo.io/docs/configuration.html
 ## Source: https://github.com/hexojs/hexo/
 
 # Site
 title: 技术资料分享小站
 subtitle: 点滴积累，细细回味。
 description: 编程技术总结与分享,记录在工作中的技术实践。
 author: Tonny Luo
 language: zh-CN
 timezone:
 
 # URL
 ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
 url: http://luohoufu.github.io/
 root: /
 permalink: :year/:month/:day/:title/
 permalink_defaults:
 
 # Directory
 source_dir: source
 public_dir: public
 tag_dir: tags
 archive_dir: archives
 category_dir: categories
 code_dir: downloads/code
 i18n_dir: :lang
 skip_render: [README.md]
 
 # Writing
 new_post_name: :title.md # File name of new posts
 default_layout: post
 titlecase: false # Transform title into titlecase
 external_link: true # Open external links in new tab
 filename_case: 0
 render_drafts: false
 post_asset_folder: false
 relative_link: false
 future: true
 highlight:
   enable: true
   line_number: true
   auto_detect: false
   tab_replace:
 
 # Category & Tag
 default_category: uncategorized
 category_map:
 tag_map:
  # Date / Time format
 ## Hexo uses Moment.js to parse and display date
 ## You can customize the date format as defined in
 ## http://momentjs.com/docs/#/displaying/format/
 date_format: YYYY-MM-DD
 time_format: HH:mm:ss
 
 # Pagination
 ## Set per_page to 0 to disable pagination
 per_page: 10
 pagination_dir: page
 
 # Extensions
 ## Plugins: https://hexo.io/plugins/
 ## Themes: https://hexo.io/themes/
 theme: jacman
 
 # Deployment
 ## Docs: https://hexo.io/docs/deployment.html
 deploy:
   type: git
   repository: https://github.com/luohoufu/luohoufu.github.io.git
   branch: master
 
 # Css Compress
 stylus:
   compress: true
 ```
  
  ### 配置jacman
  
  ``` bash
  $ git clone https://github.com/wuchong/jacman.git themes/jacman
  $ vi theme/jacman/_config.yml
  ```
 
  配置好主题相关的信息，参考：
 ``` java
 ##### Menu
 menu:
   主页: /
   归档: /archives
   简介: /about
 ##  标签: /tags
 ##  目录: /categories
 ## you can create `tags` and `categories` folders in `../source`.
 ## And create a `index.md` file in each of them.
 ## set `front-matter`as
 ## layout: tags (or categories)
 ## title: tags (or categories)
 ## ---
 
 #### Widgets
 widgets:
 - github-card
 - category
 - tag
 - links
 ##- douban
 ##- rss
 ##- weibo
   ## provide eight widgets:github-card,category,tag,rss,archive,tagcloud,links,weibo
 
 
 
 #### RSS 
 rss: /atom.xml ## RSS address.
 
 #### Image
 imglogo:
   enable: true             ## display image logo true/false.
   src: img/logo.png        ## `.svg` and `.png` are recommended,please put image into the theme folder `/jacman/source/img`.
 favicon: img/favicon.ico   ## size:32px*32px,`.ico` is recommended,please put image into the theme folder `/jacman/source/img`.     
 apple_icon: img/jacman.jpg ## size:114px*114px,please put image into the theme folder `/jacman/source/img`.
 author_img: img/author.jpg ## size:220px*220px.display author avatar picture.if don't want to display,please don't set this.
 banner_img: #img/banner.jpg ## size:1920px*200px+. Banner Picture
 ### Theme Color default is #2ca6cb
 theme_color:
     theme: '#2ca6cb'    ##the defaut theme color is blue
 
 # 代码高亮主题
 # available: default | night
 highlight_theme: default
 
 #### index post is expanding or not 
 index:
   expand: true           ## default is unexpanding,so you can only see the short description of each post.
   excerpt_link: Read More
 close_aside: false  #close sidebar in post page if true
 mathjax: false      #enable mathjax if true
 
 ### Creative Commons License Support, see http://creativecommons.org/ 
 ### you can choose: by , by-nc , by-nc-nd , by-nc-sa , by-nd , by-sa , zero
 creative_commons: none
 
 #### Author information
 author:
   intro_line1:  "爱编程的小伙"    ## your introduction on the bottom of the page
   intro_line2:  "不想做程序猿的架构师不是好的产品经理"  ## the 2nd line
   weibo: u/5666590322     ## e.g. wuchong1014 or 2176287895 for http://weibo.com/2176287895
   weibo_verifier: cd999d13    ## e.g. b3593ceb Your weibo-show widget verifier ,if you use weibo-show it is needed.
   tsina: 5666590322     ## e.g. 2176287895  Your weibo ID,It will be used in share button.
   douban: 142251737    ## e.g. wuchong1014 or your id for https://www.douban.com/people/wuchong1014
   zhihu:      ## e.g. jark  for http://www.zhihu.com/people/jark
   email: luohoufu@163.com     ## e.g. imjark@gmail.com
   twitter:    ## e.g. jarkwu for https://twitter.com/jarkwu
   github: luohoufu    ## e.g. wuchong for https://github.com/wuchong
   facebook:   ## e.g. imjark for https://facebook.com/imjark
   linkedin: luohoufu  ## e.g. wuchong1014 for https://www.linkedin.com/in/wuchong1014
   google_plus:    ## e.g. "111190881341800841449" for https://plus.google.com/u/0/111190881341800841449, the "" is needed!
   stackoverflow:  ## e.g. 3222790 for http://stackoverflow.com/users/3222790/jark
 ## if you set them, the corresponding  share button will show on the footer
 
 #### Toc
 toc:
   article: true   ## show contents in article.
   aside: true     ## show contents in aside.
 ## you can set both of the value to true of neither of them.
 ## if you don't want display contents in a specified post,you can modify `front-matter` and add `toc: false`.
 
 #### Links
 links:
   InfoQ: http://www.infoq.com/cn/,一个精英文章汇集的站点
   开发者头条: http://toutiao.io/，常去看看
   开源中国: http://www.oschina.net/,关注国内开源
   SegmentFault: https://segmentfault.com/,今天你又在开发中遇到了什么问题
 
 
 
 
 #### Comment
 duoshuo_shortname:    ## e.g. wuchong   your duoshuo short name.
 disqus_shortname:     ## e.g. wuchong   your disqus short name.
 
 #### Share button
 jiathis:
   enable: false ## if you use jiathis as your share tool,the built-in share tool won't be display.
    id:    ## e.g. 1889330 your jiathis ID. 
   tsina: ## e.g. 2176287895 Your weibo id,It will be used in share button.
 
 #### Analytics
 google_analytics:
   enable: false
   id:        ## e.g. UA-46321946-2 your google analytics ID.
   site:      ## e.g. wuchong.me your google analytics site or set the value as auto.
 ## You MUST upgrade to Universal Analytics first!
 ## https://developers.google.com/analytics/devguides/collection/upgrade/?hl=zh_CN
 baidu_tongji:
   enable: true
   sitecode: dd09496f0b593221f4326d64011cab    ## e.g. e6d1f421bbc9962127a50488f9ed37d1 your baidu tongji site code
 cnzz_tongji:
   enable: false
   siteid:    ## e.g. 1253575964 your cnzz tongji site id
 
 #### Miscellaneous
 ShowCustomFont: true  ## you can change custom font in `variable.styl` and `font.styl` which in the theme folder `/jacman/source/css`.
 fancybox: true        ## if you use gallery post or want use fancybox please set the value to true.
 totop: true           ## if you want to scroll to top in every post set the value to true
 
 
 #### Custom Search
 google_cse:
   enable: false
   cx:   ## e.g. 018294693190868310296:abnhpuysycw your Custom Search ID.
 ## https://www.google.com/cse/ 
 ## To enable the custom search You must create a "search" folder in '/source' and a "index.md" file
 ## set the 'front-matter' as
 ## layout: search 
 ## title: search
 ## ---
 baidu_search:     ## http://zn.baidu.com/
   enable: false
   id:   ## e.g. "783281470518440642"  for your baidu search id
   site: http://zhannei.baidu.com/cse/search  ## your can change to your site instead of the default site
 
 tinysou_search:     ## http://tinysou.com/
   enable: false
   id:  ## e.g. "4ac092ad8d749fdc6293" for your tiny search id
 ```
  
### 部署或更新站点

 ``` bash
 $ hexo g -d
 ```

### 创建新的博客

``` bash
 $ hexo new "blog name"
 $ hexo n page "about"
```
