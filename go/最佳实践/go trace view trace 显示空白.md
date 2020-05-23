go trace view trace 显示空白

> go trace 用来跟踪 goroutines运行情况,跟pprof配合使用，可以起到事半功倍的效果。但是，go trace 的view trace 在chrome下一片空白。

原因是因为谷歌在某一版本的chrome中禁用了本地[API](crbug.com/1036492.)

解决办法：

1. 注册一个chrome token https://developers.chrome.com/origintrials/#/register_trial/2431943798780067841 资料随便填写，记得最后一定要 feedback。
2. 修改 go trace 代码 `src/cmd/trace/trace.go` 在head中添加一行 <meta http-equiv="origin-trial" content="YOUR_TOKEN_GOES_HERE"> 然后` go install` 一下
3. 在 $gopath/misc/trace/trace_viewer_full.html 中 看到

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428161814.png)

说明配置成功，

4. 在使用 go trace得时候能够正常加载出 view trace。





Enjoy！