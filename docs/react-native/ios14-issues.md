# React Native 0.61 - iOS 14 Image 无法显示问题
> 满怀欣喜的看到苹果系统更新了，然后就知道一定或多或少用React Native写的App会出些问题。

## Issue 1: Image cannot show image in iOS 14
> 没错，图片无法显示，就是这么直接和直观。当然解决方法也比较直观。

[解决方法传送门](https://github.com/facebook/react-native/issues/29279)

个人总结：写一个fixim.sh，然后执行下
```
#!/usr/bin/env bash
echo "Fix images"
HUYDEV="_currentFrame.CGImage;"
HUYFIX="_currentFrame.CGImage ;} else { [super displayLayer:layer];"
sed -ie "s/${HUYDEV}/${HUYFIX}/" node_modules/react-native/Libraries/Image/RCTUIImageViewAnimated.m
echo "Done"
```

## Issue 2: Fast Image cannot show image in iOS 14
> 不单单是Image这个出了问题，Fast Image也无法显示了。

[解决方法传送门](https://github.com/DylanVann/react-native-fast-image/issues/702)

个人总结：pod update 下
```
cd ios
pod update SDWebImage
```
