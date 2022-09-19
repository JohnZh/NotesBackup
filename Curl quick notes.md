ref：https://www.ruanyifeng.com/blog/2019/09/curl-reference.html



# get

```sh
$ curl https://www.example.com
```



## get 查询参数 -G -d 

```sh
$ curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
```

```sh
$ curl https://tick.tmeiju.com/public/v1/service-area -G -d "receiver_lat=35&receiver_lng=139"
```

省略 -G 会变成 post



## get 查询需要 url 编码 --data-urlencode

```sh
$ curl -G --data-urlencode 'comment=hello world' https://www.example.com
```



# post -d -X

```sh
$ curl -d 'login=xxx＆password=123' -X POST https://google.com/login
or
$ curl -d 'login=xxx' -d 'password=123' -X POST https://google.com/login
```

- 使用 `-d` 参数以后，HTTP 请求会自动加上标头 `Content-Type : application/x-www-form-urlencode`
- 会自动将请求转为 POST 方法，因此可以省略 `-X POST`



## -d 读取本地文本文件的数据

```sh
$ curl -d '@data.txt' https://google.com/login
```



## 数据 url 编码 --data-urlencode

```sh
$ curl --data-urlencode 'comment=hello world' https://google.com/login
```

等价 -d，区别在于进行了 url 编码，内容的空格会编码为 20%



