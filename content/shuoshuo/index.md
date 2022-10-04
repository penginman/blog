---
title: "说说"
date: 2022-08-28T12:07:56+08:00
draft: false
comment: false
---

 
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
    ispeak
      .init({
        el: '#ispeak',
        api: 'https://kkapi.braindance.top/',
        author: '633af7214f708f8f9fb13582',
        pageSize: 10,
        loading_img: 'https://cdn.staticaly.com/gh/penginman/CDN@master/img/loading.gif',
        comment: function (speak) {
          // 4.4.0 之后在此回调函数中初始化评论
          twikoo.init({
            el: '.ispeak-comment', // 默认情况下 ipseak 生成class为 ispeak-comment 的div
            envId: "https://twikoo.penginman.com/"
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