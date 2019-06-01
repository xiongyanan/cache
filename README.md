使用Node的Express框架能够轻松的观察到这些字段带来的影响实验。接下来通过使用Express依旧保留下来的中间件express.static搭建一个建议的静态文件响应服务器。

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/1.jpg)

app.js代码如下：

```
// app.js
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');

var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public'), {
  etag: false,
  lastModified: false,
  cacheControl: false,
  // maxAge: 10000,
  setHeaders: function(res, path, stat) {
    res.set({
      expires: new Date(Date.now() + 5000)
    })
  }
}));

app.use('/', indexRouter);
app.use('/users', usersRouter);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next(createError(404));
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```
index.html 代码如下：

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    img {
      display: inline-block;
      width: 100%;
      height: 100%;
    }
  </style>
</head>
<body>
  <img src="./images/vanGogh.jpg" alt="">
</body>
<script src="./javascripts/vanGogh.js"></script>
</html>
```

先来看看expires， 如上述app.js一样把其他的缓存控制先关掉，因为他们都是默认为 true的。

然后启动服务器，访问3000端口，打开开发人员工具，你就会看到如下被种了expires的响应头了。

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/0.jpg)

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/2.jpg)

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/3.jpg)

5秒后刷新页面

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/0.jpg)
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/4.jpg)

接下来，改变app.js，

```
app.use(express.static(path.join(__dirname, 'public'), {
  etag: false,
  lastModified: false,
  // cacheControl: false,
  maxAge: 10000,
  setHeaders: function(res, path, stat) {
    res.set({
      expires: new Date(Date.now() + 5000)
    })
  }
}));
```

maxAge的单位是毫秒，此时由于优先级的关系，覆盖掉 expires 之后，在10秒之内重新刷新浏览器将不会看到资源发生变化，知道maxAge的时间到。对应的响应头如下：

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/0.jpg
)

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/2.jpg
)
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/6.jpg)

10秒后刷新页面

![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/0.jpg)

继续修改app.js:

```
app.use(express.static(path.join(__dirname, 'public'), {
  etag: false,
  lastModified: true,
  // cacheControl: false,
  maxAge: 10000,
  setHeaders: function(res, path, stat) {
    res.set({
      expires: new Date(Date.now() + 5000)
    })
  }
}));
```
因为强缓存的设置会比协商缓存被先执行，同样的操作你将看到接下来的响应头和请求头。

强缓存生效时：
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/2.jpg)

强缓存失效（10秒后刷新页面），协商缓存起作用时：
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/8.jpg)
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/9.jpg)


继续修改app.js代码，将优先级最高的 Etag 字段改为 true。


```
app.use(express.static(path.join(__dirname, 'public'), {
  etag: true,
  lastModified: true,
  // cacheControl: false,
  maxAge: 10000,
  setHeaders: function(res, path, stat) {
    res.set({
      expires: new Date(Date.now() + 5000)
    })
  }
}));
```

响应头和请求头信息如下：

强缓存生效时：
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/2.jpg)

强缓存失效（10秒后刷新页面），协商缓存起作用时：
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/8.jpg)
![image](https://raw.githubusercontent.com/xiongyanan/cache/master/public/images/10.jpg)


