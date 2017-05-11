## 集成tinker
对于原有app集成tinker，还是比较简单的，根据[tinker上的wiki](https://github.com/Tencent/tinker/wiki/Tinker-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)的指示操作即可。
具体步骤如下：
1. 在项目的build.gradle中添加 tinker-patch-gradle-plugin 的依赖
```
buildscript {
    dependencies {
        classpath 'com.tencent.tinker:tinker-patch-gradle-plugin:1.7.9'
    }
}
```
2. 然后在app的gradle文件app/build.gradle，我们需要添加tinker的库依赖以及apply tinker的gradle插件.
```
dependencies {
	//可选，用于生成application类 
	provided 'com.tencent.tinker:tinker-android-anno:1.7.9'
    //tinker的核心库
    compile 'com.tencent.tinker:tinker-android-lib:1.7.9'
}
...
...
//apply tinker插件
apply plugin: 'com.tencent.tinker.patch'
```
3. 还需要配置
```
dexOptions {
        jumboMode = true
    }
```
tinker的最佳实践，防止由于字符串增多导致force-jumbol，导致更多的变更
4. 最后可以把tinker相关的代码挪动到[tinkerpatch.gradle](https://github.com/xingstarx/TinkerDemo/blob/master/app/tinkerpatch.gradle)中
5. 改造SampleApplication，通过SampleApplicationLike代理SampleApplication的行为。具体可以看[wiki](https://github.com/Tencent/tinker/wiki/Tinker-%E8%87%AA%E5%AE%9A%E4%B9%89%E6%89%A9%E5%B1%95)和[SampleApplication](https://github.com/xingstarx/TinkerDemo/blob/master/app/src/main/java/com/star/tinker/SampleApplicationLike.java)
相关的改动可以参考 [TinkerDemo](https://github.com/xingstarx/TinkerDemo)

## 加固支持

从1.7.8开始，tinker又支持加固了，只需要修改tinkerpatch.gradle中的这部分
```
buildConfig {
            applyMapping = getApplyMappingPath()
            applyResourceMapping = getApplyResourceMappingPath()
            tinkerId = getTinkerIdValue()
            keepDexApply = false

            isProtectedApp = true //开启加固
        }
```

## 集成patchsdk
patchsdk 使用的是 https://github.com/baidao/tinker-manager/tree/master/patchsdk
步骤如下
1. 需要在app/build.gradle中添加
```
repositories {
    jcenter()
}
dependencies {
    ...
    compile 'com.dx168.patchsdk:patchsdk:1.1.3'
}
```
2. 使用ApplicationLike代理原来的Application
```
@SuppressWarnings("unused")
@DefaultLifeCycle(application = "com.dx168.patchsdk.sample.MyApplication",
        flags = ShareConstants.TINKER_ENABLE_ALL,
        loadVerifyFlag = false)
public class MyApplicationLike extends TinkerApplicationLike {
    public MyApplicationLike(Application application, int tinkerFlags, boolean tinkerLoadVerifyFlag, long applicationStartElapsedTime, long applicationStartMillisTime, Intent tinkerResultIntent) {
        super(application, tinkerFlags, tinkerLoadVerifyFlag, applicationStartElapsedTime, applicationStartMillisTime, tinkerResultIntent);
    }

    @Override
    public void onCreate() {
        super.onCreate();
        String appId = "20170112162040035-6936";
        String appSecret = "d978d00c0c1344959afa9d0a39d7dab3";
        PatchManager.getInstance().init(getApplication(), "http://xxx.xxx.xxx/hotfix-apis/", appId, appSecret, new ActualPatchManager() {
            @Override
            public void cleanPatch(Context context) {
                TinkerInstaller.cleanPatch(context);
            }

            @Override
            public void patch(Context context, String patchPath) {
                TinkerInstaller.onReceiveUpgradePatch(context, patchPath);
            }
        });
        PatchManager.getInstance().register(new Listener() {
            ...
        });
        PatchManager.getInstance().setTag("your tag");
        PatchManager.getInstance().setChannel("");
        PatchManager.getInstance().queryAndPatch();
     }
}
```
3. 搭建补丁后台管理系统
参考[Readme](https://github.com/baidao/tinker-manager/tree/master/patchserver) 搭建补丁后台，不过这个后台没有push下发补丁的功能，只有后台下发后，客户端pull拉取补丁，进行补丁修复操作
如果你的app用的是bugly来作为异常上报和运营统计，那么可以直接使用bugly提供的后台，具体操作请参考[这里](https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix/?v=20170504092424)， 小巫同学录制了一系列相关的视频，参考视频教程即可学会,地址在[这里](http://v.qq.com/vplus/bugly) ，bugly支持push下发，比自己搭建后台更加有优势，同样，也可以使用[TinkerPatch补丁管理后台](http://www.tinkerpatch.com/)，不过是收费的。

## 生成渠道包
对于渠道包，如果不是需要使用热修复，那么怎么生成渠道包都可以的。
对于flavor编译渠道包，会导致不同的渠道包由于BuildConfig变化导致classes.dex差异，这种方案是不可取的。
将渠道信息写在apk文件的zip comment中，是非常推荐的，例如可以使用项目[packer-ng-plugin](https://github.com/mcxiaoke/packer-ng-plugin)或者可使用V2 Scheme的[walle](https://github.com/Meituan-Dianping/walle), 也包括最新出来的多渠道打包神器[ApkChannelPackage](https://github.com/ltlovezh/ApkChannelPackage)，说一下区别，如果要使用热修复的话，对于不需要加固的app，那么生成渠道包，这三种方案都可以采用；对于要加固的app，只能采用ApkChannelPackage这种方案中的根据已有APK生成渠道包(如果有其他的方案，请记得告诉我)。[这篇文章](http://ltlovezh.com/2017/04/09/Android%E6%96%B0%E4%B8%80%E4%BB%A3%E5%A4%9A%E6%B8%A0%E9%81%93%E6%89%93%E5%8C%85%E7%A5%9E%E5%99%A8/)对多渠道打包工具对比做了详细的区分。目前采用的也是ApkChannelPackage方案。

## 生成补丁
以TinkerDemo为例
1. 执行./gradlew assembleRelease 生成apk
2. 使用梆梆加固工具加固apk，并签名，得到加固并且重签名的app_protected_signed.apk
3. 使用./gradlew reBuildChannel 生成渠道包
4. 修改TinkerDemo中的若干代码
5. ./gradlew tinkerPatchRelease 生成patch补丁apk(需要保证补丁包tinkerId跟基线版本tinkerId一致)(也就是说，打补丁的时候不要commit代码，tinkerId是根据git commit生成的)(也可以考虑使用每次发版本的VersionName作为tinkerId，这样处理也比较方便)
6. 如果搭建了补丁管理后台的话，使用后台上传补丁包，进行修复，根据log，可以观察到结果
7. 如果没有搭建补丁管理后台的话，使用adb push app/build/outputs/tinkerPatch/release/patch_signed_7zip.apk /storage/sdcard0/（参考tinker的sample工程，以及[tinker接入文档](https://github.com/Tencent/tinker/wiki/Tinker-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)）

## 测试补丁包
根据生成补丁的若干步骤，来测试补丁包是否有效，在第三步生成渠道包后，安装其中的一个渠道apk到手机上，然后呢，我这边是通过补丁管理后台上传的补丁，然后客户端pull补丁，根据log，可以清晰的看到补丁是否下载完成，是否有效。下载完补丁后，会进行dex合成。然后在后台或者屏幕关闭后app会被杀死，重启后补丁才会生效，这个时候我们才真正修复了问题。

## 踩过的坑
1. gradle打release包需要开启签名(./gradlew assembleRelease),不然的话，打patch包的时候，提示出错(./gradlew tinkerPatchRelease)
2. 打补丁包的时候，得保证基线版本是上一次发版本的apk，而且如果用的是tinker的默认形式的话,tinkerId得跟上一次发版本的apk的tinkerId一致，也就是基于上一次发版本的tag开bugfix分支，但是记得不要commit代码，这样就保证了tinkerId一致，如果是bugly集成的tinker，请参考bugly官方文档
3. 加固包跟渠道包如何相容的问题，一开始不太懂，[如果有加固的需求如何做?](https://github.com/ltlovezh/ApkChannelPackage/issues/2)。 我们需要搞明白的是，加固包不是基线版本，使用tinker打补丁包，用的是基线版本，得到基线版本后，我们会选择梆梆加固，乐固，爱加密这样的平台对基线版本进行加固，得到加固包后，还需要注意的是，不能够用普通的渠道包生成方案来打渠道包。举例来说吧，我司的app，采用的是梆梆加固，加固后，工具自动帮你做了生成渠道包的操作，但是这种生成的渠道包实际上是有问题的，会导致无法打补丁，无法修复，tinker热修复不生效。这种生成的渠道包，不同渠道对应的dex的CRC都不一样，我们得保证打出来的渠道包的dex都是一样的，通过apksigner可以知道梆梆加固的签名工具采用的是v1签名方案，那么我们可以考虑在APK文件的注释字段，添加渠道信息。这样就能保证不同渠道的dex是一致的。
4. [ApkChannelPackage](https://github.com/ltlovezh/ApkChannelPackage) ~~存在一点小问题，还需要作者修复，v1签名的判断逻辑有问题，加固后的apk，会改变META-INF/XXX.SF的名称,这块需要修改，不然./gradle reBuildChannel执行后，无法生成渠道包，对于开发者而言可以自己搭建本地localMaven，来测试修改这个repo~~ 作者已经修复此问题，使用1.0.4即可
5. 还有一些其他的坑已经记不得了。。。

## 参考资料
[Tinker常见问题](https://github.com/Tencent/tinker/wiki/Tinker-%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
[Android新一代多渠道打包神器](http://ltlovezh.com/2017/04/09/Android%E6%96%B0%E4%B8%80%E4%BB%A3%E5%A4%9A%E6%B8%A0%E9%81%93%E6%89%93%E5%8C%85%E7%A5%9E%E5%99%A8/)
[微信tinker补丁管理](https://github.com/baidao/tinker-manager)
[TinkerDemo](https://github.com/xingstarx/TinkerDemo)
