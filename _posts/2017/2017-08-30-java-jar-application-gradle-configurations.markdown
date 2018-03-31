---
layout: "post"
title: "java-jar-application-gradle-configurations"
date: "2017-08-30 18:07"
---

在使用gradle java plugin 编译 Java Jar 包时，容易出现一些依赖没有编译进去和一些入口配置
需要注意，gradle jave plugin 自带方法 jar，可以很方便构建 jar包。

## Manifest 配置

规则详见文档 [https://docs.oracle.com/javase/tutorial/deployment/jar/manifestindex.html](https://docs.oracle.com/javase/tutorial/deployment/jar/manifestindex.html)

```java

jar {
    manifest {
        attributes 'Main-Class':"Foo.Main",
                'Manifest-Version':'1.0',
                'Implementation-Title': 'Foo Demo',
                'Implementation-Version': version
    }
}
```

## 依赖 compile 引入

在编程时引入了 jsoniter 库， dependencies 如下：

```java
dependencies {
    compile group: 'com.jsoniter', name: 'jsoniter', version: '0.9.15'
}
```

编译执行 java -jar ... 虚拟机报 `java.lang.NoClassDefFoundError: when trying to run jar`, 现在需要配置依赖包到 jar 里面

修改jar 如下解决问题：

```java
jar {
  from {
    (configurations.runtime).collect { it.isDirectory() ? it : zipTree(it) }
  }
    manifest {
        attributes 'Main-Class':"Foo.Main",
                'Manifest-Version':'1.0',
                'Implementation-Title': 'Foo Demo',
                'Implementation-Version': version
    }
}
```

## 参考

- [DSL Reference](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.bundling.Jar.html)
- [stackoverflow](https://stackoverflow.com/questions/42458292/java-lang-noclassdeffounderror-when-trying-to-run-jar?answertab=active#tab-top)
