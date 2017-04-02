---
layout: post #post
title: Golang通过HTTP Response获取Zip文件 #post title
categories: Program #post category, seperated by spcace
tags: Golang #post tag, seperated by spcace
---

本来计划至少一周总结一次学到的东西，不过好像没有实现啊😅

一看时间都4月1号了，一个月了，总该写点啥

最近运气不好，出了很多事情，虽然都是不足为外人道的东西，但确实影响心情，希望大家每天都能开开心心的

今天写一下前几天遇到的一个需求：通过一个URL获取用户信息的zip文件

不涉及过多业务内容的话，就是要写一个小Demo，通过一个给定URL，获取一个zip文件

## 实现一个极简单的URL处理路由

既然是写Demo，那么肯定路由就Golang最简单的ListenAndServe就好了

```
func main() {
	http.HandleFunc("/zipdownload", zipHandler)
	log.Println("Listening...")
	http.ListenAndServe(":9999", nil)
}
```

## 完成zipHandler函数

按照Golang的Handler函数签名邀请，完成zipHandler函数

```
func zipHandler(rw http.ResponseWriter, r *http.Request) {
	zipName := "ZipTest.zip"
	rw.Header().Set("Content-Type", "application/zip")
	rw.Header().Set("Content-Disposition", fmt.Sprintf("attachment; filename=\"%s\"", zipName))
	err := getZip(rw)
	if err != nil {
		log.Fatal(err)
	}
}
```

### 设置响应头信息

``rw.Header().Set("Content-Type", "application/zip")``

设置response的头信息中的文件类型，对于zip文件，一般可以设置为``application/zip``或``application/octet-stream``
更具体的说明，可以在[Complete list of MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)获得，我是选择了``application/zip``来告诉服务器精确的文件类型

``rw.Header().Set("Content-Disposition", fmt.Sprintf("attachment; filename=\"%s\"", zipName))``

设置``Content-Disposition``为``attachment``即附件类型，同时设置附件文件名为我们给定的``zipName``

响应头信息中的``Content-Disposition``用于告知浏览器其获取到的文件是需要展示与页面内还是需要作为附件保存到用户本地，如果需要展示在页面内，设置为``inline``，否则设置为``attachemnt``，在设置为``attachment``时还可以额外规定该附件的文件名，更具体的说明，可以在[Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)获得

完成这两句代码后，功能其实已经完成了一大半了，最初我就是被这个地方卡住了，在获得需求时思想泡里充满了``怎么样才能让HTTP响应得到一个zip文件并且让浏览器将这个文件下载下来而不是试图展示出来``的问题

## 完成getZip函数

这里我们只需要完成一个比较基础的zip流程就好，就像我上面说的，这个需求，最难的其实是上面对HTTP Response中Header信息的了解

```
func getZip(w io.Writer) error {
	zipW := zip.NewWriter(w)
	defer zipW.Close()

	for i := 0; i < 5; i++ {
		f, err := zipW.Create(strconv.Itoa(i) + ".txt")
		if err != nil {
			return err
		}
		_, err = f.Write([]byte(fmt.Sprintf("Hello file %d", i)))
		if err != nil {
			return err
		}
	}
	return nil
}
```

### 创建zip.Writer

``zipW := zip.NewWriter(w)``

该方法创建一个``zip.Writer``，用于向zip文件中写入内容，即打包的文件

参数为``io.Writer``，那么我们这里当然就是使用``http.ResponseWriter``

返回值为一个``zip.Writer``，最后的zip内容都会写入这个``zip.Writer``，而最终当然是写入了参数的``io.Writer``中，也就是我们的``http.ResponseWriter``中

***记得``defer zipW.Close()``关闭``zip.Writer``***

### 向zip.Writer中写入文件

``f, err := zipW.Create(strconv.Itoa(i) + ".txt")``

该方法向``zip.Writer``中添加一个文件，也就是说向zip文件中添加一个文件

参数字为字符串，会作为写入zip中的文件的文件名

第一个返回值为一个``io.Writer``，用于向其中，也就是向我们添加到zip的文件中，写入文件内容，即如``_, err = f.Write([]byte(fmt.Sprintf("Hello file %d", i)))``代码所示，我们向文件中写入了简单的字符串

## 运行程序

接下来，只需要在程序``main.go``所在目录运行``go run main.go``就可以在你的浏览器访问``localhost:9999/zipdownload``了，浏览器会下载一个zip文件

![下载的ZipTest及解压后的内容](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-01-post01/ZipTest_zps5m0dywev.png)

最后，代码已经上传到了我的[Github仓库](https://github.com/kenshinsyrup/AllGolangDemo/blob/master/GetZipFromHTTP/main.go)可以直接下载测试





