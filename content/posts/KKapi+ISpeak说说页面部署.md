---
title: "KKapi+ISpeak说说页面部署"
categories: ['瞎折腾']
tags: ['说说']
date: 2022-10-04
lastmod: 2022-10-16

---

## 前言
感觉原来的 Artitalk 说说不好康，在开往里发现好多博客都用的说说功能叫叨叨点啥，看了看作者的[说说页面](https://www.antmoe.com/speak/)，可以插入图片和标签分类，还有仅自己可见的功能，感觉挺不错的（实际是自己想折腾）所以就整一个。自己在部署过程中实在是踩了不少的坑，而且作者的文档感觉写的也不算完善，所以打算自己记录一下。

## 项目结构
作者的文档中各种仓库链接属实给我跳晕了，最后理出来的项目分为以下部分：
* `kkapi`。是作为说说的后端部分，连接 MongoDB 数据库，还有一个`kkadmin`的管理页面
* `ISpeak`。说说的主体部分，依赖于后端的 `kkapi` ，分为前端的展示页面，和一个对接后端的发布说说页面。

作者文档中给出很多部署方法，白嫖版的就是 vercel 后端 api + 管理界面 + MongoDB 提供的云服务，但是个人感觉 vercel 经常被墙，所以部署的 api 感觉也不会稳定，而且考虑到数据的存放问题，所以我选择的是都部署到自己服务器上。

## 后端部署

 ### Docker 安装 Mongodb

安装可以参考菜鸟教程的 [Docker 安装 MongoDB](https://www.runoob.com/docker/docker-install-mongodb.html) 。因为之前听过 MongoDB 的未授权访问，所以考虑到安全性问题，创建容器的时候添加 `MONGO_INITDB_ROOT_USERNAME` 和 `MONGO_INITDB_ROOT_PASSWORD` 设置用户的账号密码，开启Docker MongoDB 的身份验证。考虑到数据未来的迁移可以通过 `-v` 挂载宿主机的一个目录。可以修改默认端口再减少一些风险。最后我启动的命令如下 

```shell
docker run -d --name mongodb \
	-p xxxxx:27017 \
	-v /my/own/datadir:/data/db \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	--restart=always
	mongo
```

之后可以使用工具测试一下连接。

### kkapi 部署

和项目文档中的教程差不多，要注意使用的 node 版本请高于 `16.0.0`
1. 首先克隆项目源码
`git clone https://ghproxy.com/https://github.com/kkfive/kkapi-open.git`
2. 接下来项目需要安装的工具 `yarn` 和 `pm2`，分别是
`npm i yarn -g`
`npm i pm2 -g`
3. 然后安装项目所需依赖 `yarn install` 。
4. 之后再执行 `yarn build` 编译项目。这里我的小鸡顶不住编译所以自己在本地编译传上去了💧。
5. 在项目文件夹创建环境变量文件，格式如
```env
PORT=3000
DATABASE_URL=mongodb://127.0.0.1:27017/kkpaiopen?authSource=admin
DATABASE_USER=mongoadmin
DATABASE_PASSWORD=secret
# 加密密钥 测试
SECRETKEY=xxxxxxxxxxxxxxx
```

这里的数据库连接地址我原来还想使用MongoDB提供的免费云服务当数据库，但是没搞成功，所以最后使用了本地的 MongoDB，有大佬知道的可以指点一下。

6. 使用 `pm2` 使用守护线程启动项目
`pm2 start pm2.json`

我启动项目遇到了 `[PM2][WARN] Expect “restart_delay” to be a typeof [object Number], but now is [object String]` 错误，这个错误原因是作者的  pm2.json 中的 `restart_delay` 值是字符串类型 `60s` 改成数值 `60` 就可以了。

7. 测试项目是否成功启动
可以使用 `lsof -i:端口` 查看端口是否被监听判断项目是否成功启动。没成功的原因大概率是因为数据库连接地址、数据库账号密码不正确。

8. 创建初始化用户
`curl http://127.0.0.1:3000/api/user/init`
创建的默认用户名和密码是 `admin` 和 `123456`，这个用户名密码用来登陆可视化的管理后台，并且用户似乎**只能拥有一个**。

### kkapiadmin（可视化管理后台）

参考[官方文档](https://kkapi.js.org/guide/admin/setup.html)中的教程，使用的 Vercel 部署的。这个墙不墙的就无所谓了，注意的坑有：
* 修改部署分支和生产分支为 `vercel`。
* fork 作者仓库的时候记得把 only fork master 取消勾选。

之后登录就是用前面初始化的用户名密码，进入后台以后可以修改密码。登陆后台以后需要设置：
* ISpeak 标签。因为发布说说是需要选择标签的，标签中的背景颜色值是**十六进制的颜色**代码
* 添加用户token。**需要注意！！！**，添加的token的**标题**只能是 `speak` 不能是其他的，否则发布说说时会提示token不存在，发布时验证的就是字段为 `speak` 的token的值。

![](https://img.braindance.top/article/2022/10/04/c5191febc049fbed86f5b77df8367c89.png)

接下来可以在前端说说页面测试发布说说，发布说说需要输入后端 kkapi 地址、用户id （在管理后台可以找到）、token。网址：https://ispeak-biubiu.vercel.app/

![](https://img.braindance.top/article/2022/10/04/778dcc5fe051722e4f9a919b7a9e2a61.png)

![](https://img.braindance.top/article/2022/10/04/491fff2969d731ff17d8799fe6a20d14.png)

发布成功可以在后端看到发布的说说。

## 前端部署
我使用的是 Ispeak 搭配的 twikoo 评论，因为现在博客使用的就是 twikoo，省去了再部署评论的麻烦。根据[ISpeak文档部分](https://kkapi.js.org/posts/ispeak/)，[ispeak 配置项](https://github.com/kkfive/ISpeak/blob/master/src/types/parameter.ts)中 `comment` 是一个回调函数，可以自行初始化评论，参照twikoo评论初始化的格式。我博客中的说说页面代码

```html
<div id="tip" style="text-align:center;">ipseak加载中</div>
<div id="ispeak"></div>
<link
  rel="stylesheet"
  href="https://cdn.staticfile.org/highlight.js/10.6.0/styles/atom-one-dark.min.css"
/>
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/ispeak@4.4.0/style.css"
/>

<script src="https://cdn.staticfile.org/highlight.js/10.6.0/highlight.min.js"></script>
<script src="https://cdn.staticfile.org/marked/2.0.0/marked.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/ispeak@4.4.0/ispeak.umd.js"></script>


<script src="https://cdn.staticfile.org/twikoo/1.6.7/twikoo.all.min.js"></script>
<script>
  var head = document.getElementsByTagName('head')[0]
  var meta = document.createElement('meta')
  meta.name = 'referrer'
  meta.content = 'no-referrer'
  head.appendChild(meta)
  if (ispeak) {
    ispeak.init({
        el: '#ispeak',
        api: '这里是后端kkapi地址',
        author: '后端用户id',
        pageSize: 10,
        loading_img: 'https://bu.dusays.com/2021/03/04/d2d5e983e2961.gif',
        comment: function (speak) {
          const { _id, title, content } = speak
          // 4.4.0 之后在此回调函数中初始化评论
          //这里是twikoo的初始化配置，如果使用其他评论可以在这里修改
          twikoo.init({ 
            el: '.ispeak-comment', // 默认情况下 ipseak 生成class为 ispeak-comment 的div
			path: '/shuoshuo/?q=' + _id,
            envId: "twikoo后端地址"
          })
        }
      })
      .then(function () {
        console.log('ispeak 加载完成')
        document.getElementById('tip').style.display = 'none'
      })
  } else {
    document.getElementById('tip').innerHTML = 'ipseak依赖加载失败！'
  }
</script>
```

更新一波。被人发现了说说的评论没有独立，自己改了下配置。
上面的代码加入了 32 和 37 行代码，其中 37 行 `path` 属性设置为你当前的说说页面路径加 `q` 参数，这个参数可能无所谓吧，但是 `_id` 是当前说说的唯一 id，因为自己在页面中测试时，说说评论请求的地址格式也是根据 37 行代码这个进行请求查询的。

## Github 登陆验证（可选*）
可以发布仅登陆可见的说说，但是需要配置 Github app。
1. 参考[项目文档](https://kkapi.js.org/guide/setup/github.html)创建 app ，其中填写的 speak 页面路径就是 ISpeak 所在的博客路径
2. 创建以后拥有了 `Client ID` 和 `Client Secrets`，这两项需要填写在 kkapi 后端部署的 `local.env` 配置中。
3. 在 kkapi 的后端界面个人设置中填写 `GitHubId` 。获得方法访问 github 提供的接口
`https://api.github.com/users/<Your UserName>`注意替换尖括号整体为你github的用户名，不是昵称。

![](https://file.acs.pw/picGo/2022/03/13/20220313121930.png)

4. 在前端页面的 `speak` 初始化中添加两个属性
```js
ispeak.init({
...
speakPage："/shuoshuo/",  //这里是说说的页面路径，对应于 github app 中填写的 speak 页面路径（用双引号括起来，我不知道为啥单引号不行）
githubClientId: 'Iv1.*******',  //github app 的 Client ID
...
})

```

然后就可以在你的说说下面找到一个 Github 授权登陆。

![](https://img.braindance.top/article/2022/10/04/4aeea0532e5dc44c83a6822033d9971e.png)

## 完工
说说还支持 markdown 格式的图片插入，看起来更好用了，给作者点个赞。