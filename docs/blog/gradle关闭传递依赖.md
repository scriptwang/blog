- 场景
本项目的`build.gradle`依赖于`commons`，本项目需要`ojdbc8`，但是`commons`里面已经有了`ojdbc6`，现在想要在不更改`commons`的情况下让本项目使用`ojdbc8`（因为`commons`是公用的，更改了可能导致别人出现问题）可以使用关闭传递依赖选项`transitive = false`

参考：https://stackoverflow.com/questions/33926800/how-can-i-exclude-dependencies-brought-in-from-other-sub-projects
```groovy
dependencies {
    compile (project(':commons')){
        //关闭传递依赖，即只依赖commons本身，不依赖commons里面的包
        transitive = false
    }
    compile(
            //excel
            "org.apache.poi:poi-ooxml:3.14",
            ...
    )

}
```