---
layout: post
title: [node]request+watch开发自测的懒人神器
category: life
tags: [life]
---




## 序

关于自测的重要性,此处省下1万字,简而言之， 如果不想让 QA 小姐姐把 BUG 甩你一脸,那你就得学会高效自测。

另外呢，有一些水平很差的程序猿，你不让他有一个自测的环境，可以自己跑着，不停的 curl 着，不停的 postman着，他就不会编码---比如本拐。

但是无论 postman，还是curl，总是要费时间的，你要不停的去回车，去试————对于程序（懒）猿（人）来讲，任何重复的劳动都是不能忍受的.

在这里，向大家极力推荐 node.js ，因为两个重要的 npm 库，一个是 node-watch ,可以监测文件的变成，在文件有改变时发生相应的事件。 另一个呢,则是 request ,一个优秀的 http client 库，借助这两点，我们可以在写一个简单的 js脚本，当源码文件发生改变时，自动去调用相应的接口。

## 思路及源码

知道了几个库，还远远不够，我们想做的时，写一个简单的框架，可以将我们的case配置进去。 同时可以进行一些参数的传递。 那么，一个配置，我希望他是这样的:

1.  `module.exports = {`

2.  `type: 'lua',`

3.  `dir: '/data/guaiye/user/',`

4.  `host: 'http://localhost:8080',`

5.  `cases: [`

7.  `{`

8.  `name : '请求登录验证码',`

9.  `index: 1,`

10.  `key : 'req',`

11.  `uri: '/v1/auth/require-checkcode',`

12.  `method: 'post',`

13.  `data: {`

14.  `mobile : '13800000001',`

15.  `source: 'testApp'`

16.  `},`

17.  `},`

18.  `{`

19.  `name : '验证码登录',`

20.  `index: 2 ,`

21.  `key : 'check',`

22.  `uri: '/v1/auth/login-checkcode',`

23.  `method: 'post',`

24.  `data: {`

25.  `mobile : '13800000001',`

26.  `code: '#req.res.data',`

27.  `source: 'testApp'`

28.  `}`

29.  `},`

30.  `{`

31.  `name : '验证token',`

32.  `index: 3,`

33.  `key: 'info',`

34.  `uri: '/v1/auth/info-by-token',`

35.  `method: 'post',`

36.  `method: 'post',`

37.  `data: {`

38.  `source: 'testApp',`

39.  `token : '#check.res.data.token',`

40.  `}`

41.  `},`

42.  `{`

43.  `uri: '/v1/auth/logout',`

44.  `name : '登出',`

45.  `method: 'post',`

46.  `data: {`

47.  `token : '#check.res.data.token',`

48.  `}`

49.  `}`

50.  `]`

51.  `}`

那么，在**希望可以按如此进行配置case**的基础上，我们去写一个可以遍历这个case的脚本。 首先，简单封装一下request库：

1.  `const request = require('request')`

2.  `let execReq = async(opt)=>{`

3.  `return new Promise((resolve,reject)=>{`

4.  `request(opt,(err,res,body)=>{`

5.  `if (err){`

6.  `reject(err)`

7.  `}else{`

8.  `resolve(body)`

9.  `}`

10.  `})`

11.  `})`

12.  `}`

13.  `let getReq = async(url) =>{`

14.  `return await execReq({`

15.  `url:url,`

16.  `method:'GET',`

17.  `json:true,`

18.  `headers:{`

19.  `'content-type':'application/json'`

20.  `}`

21.  `})`

22.  `}`

23.  `let postReq = async (url,data)=>{`

24.  `return await execReq({`

25.  `url:url,`

26.  `method:'POST',`

27.  `json:true,`

28.  `body:data,`

29.  `headers:{`

30.  `'content-type':'application/json'`

31.  `}`

32.  `})`

33.  `}`

34.  `module.exports = {`

35.  `exec:execReq,`

36.  `get:getReq,`

37.  `post:postReq`

38.  `}`

在这个思路的基础上，写一个简单的app.js

1.  `const cfg = require ( './config') //我们的配置文件`

2.  `const watch = require('node-watch') // watch神器`

3.  `const _ = require('lodash')`

4.  `const http = require('./tools/http') // 上面封装的http`

5.  `const log = (msg) =>{`

6.  `console.log(new Date().toLocaleTimeString()+':'+msg)`

7.  `}`

8.  `const run_test = async()=>{`

9.  `log(`the .${cfg.type} file in ${cfg.dir} changed,run the test`)`

10.  `log(`the host is ${cfg.host}`)`

11.  `let all = {}`

12.  `for(let i = 0 ; i < cfg.cases.length; i ++)`

13.  `{`

14.  `let testCase = cfg.cases[i]`

15.  `let url = cfg.host + '/'+testCase.uri`

16.  `let method =testCase.method || 'get'`

17.  `let postdata = _.cloneDeep(testCase.data || {})`

18.  `all[testCase.key]={}`

19.  `try{`

20.  `for( let p in postdata){`

21.  `let str = postdata[p]`

22.  `if (str.indexOf('#') == 0 ) {`

23.  `let key = str.substr(1)`

24.  `postdata[p] = _.get(all,key)`

25.  `}`

26.  `}`

27.  `let data = await http[method](url,postdata)`

28.  `all[testCase.key].req = postdata`

29.  `all[testCase.key].res = data`

30.  `console.log({name:testCase.name,url,method,req:postdata,res:data})`

31.  `}catch(e){`

32.  `console.log(e)`

33.  `}`

34.  `}`

35.  `}`

36.  `watch(cfg.dir , {recursive: true,filter : new RegExp(`\.${cfg.type}$`)},run_test)`

37.  `run_test()`

在这里用了一个小把戏，就是req的定义如果以 '#' 开始，则从测试的上下文中进行取值。
也是用了现成的lodash库。

OK，现在为止，大功告成，每当接一个新需求，我会配置一份测试文件，然后让自己的测试脚本跑进来，时刻看着代码的调试情况。

这个东西虽然简单，其实可以扩展的无限灵活，全当是抛砖引玉，给大家一个思路了。

