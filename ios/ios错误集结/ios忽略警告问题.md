# ios忽略警告问题

可以根据网址中的警告类型来填写需要忽略的问题
```      
       #pragma clang diagnostic push
       #pragma clang diagnostic ignored "-Wincompatible-pointer-types"
       [_controllerDelegate yztwebViewDidStartLoad:webView];
       #pragma clang diagnostic pop
```

![image.png](http://upload-images.jianshu.io/upload_images/1401554-536386ccaba4ee7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[忽略警告问题](http://fuckingclangwarnings.com/)
