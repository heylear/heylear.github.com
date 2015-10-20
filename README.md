# 博客地址

<http://www.heylear.com/>

# 必改内容

## 1.swiftype

此服务提供站内搜索功能

服务地址：<https://swiftype.com/>

设置方法可参考 <http://opiece.me/2015/04/16/site-search-by-swiftype/>

设置完毕后，您需要修改 `_config.yml` 中 `swiftype_searchId`。

在自己的引擎中，进入 `Setup and integration` -> `Install Search`, 你将找到 `swiftype_searchId`。

```html
<script type="text/javascript">
...
...
  _st('install','swiftype_searchId','2.0.0');
</script>
```

## 2.畅言

此服务提供评论功能

服务地址：<http://changyan.kuaizhan.com/>

设置方法可参考 <http://changyan.kuaizhan.com/static/help/>