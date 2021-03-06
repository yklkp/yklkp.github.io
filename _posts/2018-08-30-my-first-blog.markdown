---
layout: post
title: 千里之行，始于足下
date: 2018-08-30 16:06:20 +0500
img: i-rest.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [杂项]
---
作为一个程序员，时常总结是个好习惯，知识的海洋是如此之大，要想学好一门技能，勤奋，反思，总结，是不可少的步骤。
高中之前，还是少年的我，对文字有着不一样的感觉，总觉得自己以后会成为一个作家之类的，文笔还是有那么点附有感染力，灵感也是不断涌现。自从高中选择了理科，我知道对于文字的一腔热爱也终将被现实覆盖，直到现在毕业工作了几个月后，偶然在网上看到一位资深程序员写的一篇文章，才发现原来程序员也可以有着自己的爱好，除了代码之外的爱好，写字，创作。或许对于现在的我来说，这种爱好的发展空间并不会很大，因为刚出社会，我需要花大量的时间去学习技能，社会生存之道。如此看来，每次的学习总结，也就是唯一能够与未来的我交流的时光了。当然，在闲暇时候，若是想写些生活感悟，情感宣泄，还是可以记录下来的。
这是我在github上刚搭建的个人博客。接下来我需要做的是将博客的标签分为几类（PHP、JS、Jquery等等）。在此之前，我想先记录一下搭建此博客的方法与步骤。

## 如何在Github上搭建个人博客
1. 准备一个github账号
2. 下载git
3. 在github上创建新仓库，方法有多种，这里我只总结两种：一个是在github上直接创建，第二种是使用phpstrom来快速创建。
    - 方法一：
![图一]({{site.baseurl}}/assets/img/2018/new1.png)
![图二]({{site.baseurl}}/assets/img/2018/new2.png)
    这里需要注意的是图二中，那个username就是自己github账号的名字，仓库名就是后面自己访问博客的域名。
    - 方法二：在phpstrom中的 VCS-----Import into version control----Share Project on GitHub，然后输入自己的github账号与密码，登陆成功后，再输入仓库名，名字和方法一中一样，需要自己的名字加上.github.io，这样才可以通过这个域名访问到自己的博客。
4. 创建SSH KEY，这个可以通过本地安装的git工具快速获取，鼠标右击，选择Git Gui Here，然后点help，Show SSH Key，中间需要填什么的都可以不填，一路下去，可见下图
![gui]({{site.baseurl}}/assets/img/2018/new4.png)

然后在github上面的setting中的Deploy keys里添加上复制过来的keys。
![图四]({{site.baseurl}}/assets/img/2018/new3.png)
### Jekyll的使用
5. 使用Jekyll下载模板。[Jekyll网址](http://jekyllthemes.org/)
6. 下载下来的模板使用需要注意以下几点：
    - 每个博客文章放入_posts文件夹下，并命名格式必须是时间加名称，如:2018-08-30-my-first-blog。
    - _config.yml是全局配置文件，需要注意的是将base_url中改为空，理由：在我们一路创建的仓库中，模板所引用的路劲是根目录下的，访问项目的时候所找的路径也是从根目录下开始找的，所以这里base_url为/或空。若是我们在此域名下所建多个项目，如blog1、blog2等，则其中的base_url就要匹配所对应的博客名字，"/blog1"，"/blog2"。
7. 做完以上步骤即可通过域名访问到自己的博客了。下面我们可以在此博客中加入一些其他的网页链接，比如加上自己的简历，简历的模板样式等需要单独放入一个文件夹中，避免混乱。所要注意的是点击外连接的html要放在根目录下，即和index.html同级下，因为我们访问的时候是域名+文件夹名字。（当然也可以在a标签中设置其他路径，为了样式引入方便，则选择放入根目录下）

![end]({{site.baseurl}}/assets/img/we-in-rest.jpg)

## 如何使用phpstrom管理自己的博客
- 首先，本地是需要装git工具，然后在settings中的version control里的git设置git.exe的位置。在自己安装git的位置里面的cmd里。
- 然后点击VCS，再点Checkout From Version Control，再点git。将自己的博客github地址填上，即可pull下来。
- 每次提交前需要先pull一下，这个是在团队合作中必须要做的一步，防止和别人的代码冲突覆盖。当然我们这里自己的博客不需要这一步，当然，如果你有两个地方修改的时候还是需要pull下来的，比如在家和公司。总之，如果只是在一台电脑上操作则不需要pull，若是一台以上必须要先pull。提交push时候要先commit，写上提交文字说明，最后push。
- 若是直接在本地粘贴复制过来的代码或图片的时候，可以发现其颜色是红色的，则这个时候需要右击，然后Add，这样就变成绿色了。
