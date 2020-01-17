# BitCode

问题：
Bitcode在链接静态库时的问题处理 “ld: bitcode bundle could not be generated because”

bitcode官方文档：https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f

Bit code据官方介绍是一种中间代码，苹果会进行优化等处理，遇到编译问题直接关闭的话是能解决问题，但是个人觉得不是很好。

解决方法：

```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'YES'

      if config.name == 'Release'
          config.build_settings['BITCODE_GENERATION_MODE'] = 'bitcode'
      else
          config.build_settings['BITCODE_GENERATION_MODE'] = 'marker'
      end

      cflags = config.build_settings['OTHER_CFLAGS'] || ['$(inherited)']

      if config.name == 'Release'
          cflags << '-fembed-bitcode'
      else
          cflags << '-fembed-bitcode-marker'
      end

      config.build_settings['OTHER_CFLAGS'] = cflags
    end
  end
end
```

参考文章：
https://www.jianshu.com/p/41580e642b9e
