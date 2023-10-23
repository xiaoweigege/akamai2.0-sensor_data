
## 1. 引言
Akamai反爬虫系统是保护网站避免遭受攻击以及api数据安全的一套产品。目前他的防护手段有两方面，一个是在https请求层面的防护，另外一个是收集当前浏览器信息，进行一个风控拦截。不同站点，不同接口风控拦截的程度都不同。

## 2. 关于https请求层面ja3指纹的绕过
目前网络上有很多关于过ja3的请求文章。推荐一个Python第三方过ja3的库 [curl_cffi](https://github.com/yifeikong/curl_cffi) 

## 3. sensor_data风控验证流程
本次分析的站点是[maersk.com](https://www.maersk.com/)站点。主要的验证是通过 **_abck** 这个cookie值来进行校验的。提交 **sensor_data** 参数返回过来的。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685808309127-afadf7a0-2646-4760-8243-6dcb9981a205.png#averageHue=%23777d85&clientId=ud24bafdf-8053-4&from=paste&height=528&id=u7a468abe&originHeight=1056&originWidth=2280&originalType=binary&ratio=2&rotation=0&showTitle=false&size=772599&status=done&style=none&taskId=u18ace2ff-0f00-4f4f-9341-e06ff92d2f1&title=&width=1140)
如果 **sensor_data** 这个值合法会返回一个有效的 **_abck**。接下来就可以愉快的请求对应网站的内容了。不过 **_abck** 是有时效的。被403拦截之后就需要重新请求生成。

## 4. sensor_data 参数分析
现在我们去分析 **sensor_data **这个参数是如何生成的。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685808732764-3bfa0b39-494c-4465-ad1f-fa60d81ab428.png#averageHue=%23232428&clientId=ud24bafdf-8053-4&from=paste&height=669&id=u2b53119f&originHeight=1338&originWidth=2242&originalType=binary&ratio=2&rotation=0&showTitle=false&size=538240&status=done&style=none&taskId=ua2c16877-96a8-46e1-8dc5-35d0a8be7c9&title=&width=1121)
喏，瞧就是这个js生成的，混淆的很恐怖。不过也没关系，可以利用**AST**技术进行还原。这块还原知识点可以了解一下蔡老板的AST知识星球。不过我没有进行还原。硬着头皮进行调试。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685808967444-dff2a27e-b378-44b8-b956-ec6a54f2518b.png#averageHue=%23242629&clientId=ud24bafdf-8053-4&from=paste&height=564&id=u865e9387&originHeight=1128&originWidth=1756&originalType=binary&ratio=2&rotation=0&showTitle=false&size=378413&status=done&style=none&taskId=u2ab36a67-be33-44e2-93d8-c7a587fb5c7&title=&width=878)

可以看到 **sensor_data **参数是58长度的数组进行拼接加密生成的。我们接下来具体分析一下这些数组中的内容是什么。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685809243314-a6ab8099-112f-4ccb-a729-f912bcc393fd.png#averageHue=%23202225&clientId=ud24bafdf-8053-4&from=paste&height=832&id=u426dc28e&originHeight=1664&originWidth=1930&originalType=binary&ratio=2&rotation=0&showTitle=false&size=353497&status=done&style=none&taskId=uf7e8705c-cfbb-4c8c-8898-cd46d32cdef&title=&width=965)
这些是数组的具体内容。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685809090261-e3dff937-bc1c-4935-a77f-9dfded1fda03.png#averageHue=%232d2d2d&clientId=ud24bafdf-8053-4&from=paste&height=509&id=u5c924a7d&originHeight=1018&originWidth=1512&originalType=binary&ratio=2&rotation=0&showTitle=false&size=387913&status=done&style=none&taskId=u5f5d738d-9f0b-4a0f-a77d-76cd3d6d941&title=&width=756)
这是我经过调试他的js代码进行分析得出的一些具体参数的含义。
其中通过我的分析和测试 **canvas**指纹和**运动轨迹**是最重要的，**更关键的是canvas指纹还不能随机伪造**。canvs指纹可以去收集一些真实浏览器的来进行替换。那么运动轨迹的话，可以写算法模拟真实的鼠标移动、点击等等行为。

:::info
###### 高并发的要求是模拟出来的浏览器指纹信息和运动轨迹数据要足够真实，并且需要动态更换ip，一个ip高并发请求会被拦截掉。还有一个关键信息是生成的参数不够真实的话akamai系统会给通过，但是会拖慢请求接口的时间！！在一些重要性的接口中会体现出来。
:::
## 5. akamai-bm-telemetry参数生成
该参数是在接口中的headers中带的。他生成的方式就比较简单了，是基于 **sensor_data** 参数进行生成的，用了base64进行编码组成的。我们来看一下
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685810544615-ef1ab010-6d7b-4375-ad49-509d6c38acff.png#averageHue=%23242629&clientId=ud24bafdf-8053-4&from=paste&height=206&id=u5ed7951e&originHeight=412&originWidth=2018&originalType=binary&ratio=2&rotation=0&showTitle=false&size=156618&status=done&style=none&taskId=u44dd2324-2b55-4c63-9fc4-eb4d28816dd&title=&width=1009)

经过代码翻译
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685810585928-04c403de-2bc8-496f-a9b5-064bfa8db473.png#averageHue=%232e2d2b&clientId=ud24bafdf-8053-4&from=paste&height=56&id=ub28593d6&originHeight=112&originWidth=1434&originalType=binary&ratio=2&rotation=0&showTitle=false&size=38021&status=done&style=none&taskId=u3ee59c0f-f7ba-46e6-a16a-1fc832a782e&title=&width=717)
具体就是这么生成的。

## 6. 成果展示
![image.png](https://cdn.nlark.com/yuque/0/2023/png/34896050/1685810646236-8a658740-246d-4f1d-98de-85cbf3f66e95.png#averageHue=%234b4949&clientId=ud24bafdf-8053-4&from=paste&height=934&id=u7dc07597&originHeight=1868&originWidth=1806&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1297220&status=done&style=none&taskId=u2aa30fa7-2e49-48e3-a3d3-00c27a36c85&title=&width=903)
100个并发通过率100%
可以看看这个通过失效，生成比较真实的 **sensor_data** 通过时效就很高！一秒内就能过掉**akamai**的系统。如果置信度不是很高的话会拖慢速度10s内才通过。当然这跟ip也有一定的关系。

## 7. 商务合作
生成 **sensor_data** 的算法我已经封装成api了，可以进行调用，如果不想自己研究的话，可以联系我进行接口调用服务。当然这是要收费的。
我的联系方式 

wechat: **King_ww__**

qq: **240942649**

telegram: [xwggnp](https://t.me/xwggnp) 

