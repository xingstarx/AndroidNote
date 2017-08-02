## gradle常用命令

1. debug编译并安装 ./gradlew installDebug
2. release编译并安装 ./gradlew installRelease
3. 生成debug包 ./gradlew assembleDebug
4. 生成release包 ./gradlew assembleRelease (./gradlew assemble直接生成这两个)
5. android studio lint检查代码和资源文件 ./gradlew lint
6. clean 工程编译生成的class等资源文件 ./gradlew clean
7. 重新构建 ./gradlew build(重新构建，特别耗时，在我开发过程中，一般不建议这么用，而是用./gradlew installDebug)
8. 下载没有依赖完的库 gradle build --refresh-dependencies (这个命令有点耗时，会跳过缓存重新下载依赖)(记得不是很清楚了)
9. 查看依赖的几种方法 
	1 在terminal里输入gradlew :app:dependencies --configuration compile其中app换成你想要看依赖的module的名称	2 在工程的build.gradle中追加这段代码		subprojects {    	task allDeps(type: DependencyReportTask) {}	}	然后再terminal里输入gradlew allDeps就行了
	3 Android studio安装个插件plugin, gradle view
