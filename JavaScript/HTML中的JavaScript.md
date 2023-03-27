# HTML中的JavaScript

## <script>元素

浏览器在解析HTML 的时候，遇到一个没有任何属性的标签，就会暂停解析，先发送网络请求获取该脚本的内容，然后让JS引擎执行该代码，当代码执行完毕后恢复解析。

有下列常用属性：

- async: 立即下载脚本，但是不会阻塞浏览器解析文档，一旦网络请求回来之后，如果此时HTML还没有解析完，浏览器则会先暂停解析，先让JS引擎执行脚本代码，执行完毕之后恢复解析，如果存在多个async，它们之间的执行顺序也是不定的，谁先请求到资源谁先执行，只能用于外部链接。
- defer:立即下载脚本，但是不会阻塞浏览器解析文档，一旦网络请求回来之后，如果此时HTML还没有解析完，则会等待HTML解析完成之后再执行JS代码，如果存在多个defer，则会按照在文档中出现的顺序进行执行，不会破坏JS之间的依赖性，只能用于外部链接。
- integrity: integrity属性可以设置文件的hash值，浏览器下载JS文件的时候会对文件进行hash计算，如果一致则加载，不一致则拒绝加载，一般用于内容CDN等内容不会被恶意篡改。
- crossorigin：crossorigin的属性值可以是anonymous或者use-credentials，如果没有属性值或者非法属性值，则默认是anonymous，crossorigin会让浏览器启用CORS访问检查，检查http相应头的Access-Control-Allow-Origin，对于传统script需要跨域获取的JS资源，控制暴露出其报错的详细信息。
- type： 表示代码块中脚本语言的内容类型（MIME类型），如果是module，则表示代码会被当成ES6模块，能够使用import/export