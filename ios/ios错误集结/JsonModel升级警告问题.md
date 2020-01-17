# JsonModel升级警告问题


##1 说明
initWithDisctionary方法已经deprecated
需要使用initWithModelToJSONDictionary代替
而且dictionary的写法和之前相比正好反了一下
现在是model的属性名作为key 而json里面对应的属性作为value

##2 产生的问题
一大堆警告，有时候报错都得翻半天才到底

##3 解决措施

> 正则公式!!!：
Replace: 
搜索方案1： `@"(.*)".*@"(.*)".*`  
搜索方案2： `@"(.*)"\s*:\s*@"(.*)" ` 匹配中间带冒号已经空格的字符串，更精确
With: `@"$2":@"$1"`

cmd + f 搜索，点击replace，输入正则，替换key value如图：
![处理图片](http://upload-images.jianshu.io/upload_images/1401554-fe7b5a06bd8d68ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4 注意的问题
不要把其他的请求参数替换，要小心、