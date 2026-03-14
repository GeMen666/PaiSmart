## Windows 安装 Elasticsearch 8.10 踩坑总结

记录一次在 Windows 环境从零安装 Elasticsearch 8.10 的常见问题与解决方案。

------

### 1. `InvalidPathException` 启动失败

**报错**

```
java.nio.file.InvalidPathException: Trailing char < >
```

**原因**

系统环境变量 `CLASSPATH` 包含非法路径或旧版 Java 路径，例如：

```
%JAVA_HOME%\lib\dt.jar
%JAVA_HOME%\lib\tools.jar
```

这些是 **JDK8 时代的文件**，JDK9 之后已经不存在。

**解决**

删除系统环境变量：

```
CLASSPATH
```

或清空其中的 `dt.jar / tools.jar`。

------

### 2. `NoSuchFileException: dt.jar`

**报错**

```
NoSuchFileException: ...\jdk-17\lib\dt.jar
```

**原因**

同样是旧的 `CLASSPATH` 指向 JDK8 文件。

**解决**

删除 `CLASSPATH` 环境变量即可。

------

### 3. WSL 与 Windows 环境不互通

在 WSL 中运行 Windows 版 Elasticsearch 会报：

```
jdk/bin/java: No such file or directory
```

**原因**

WSL 不能直接运行 Windows 二进制程序。

**解决**

两种方案二选一：

- Windows 版 ES → 在 Windows 终端运行
- Linux 版 ES → 在 WSL 中安装

本次选择 **Windows 运行 ES**。

------

### 4. PowerShell `curl` 参数冲突

**报错**

```
参数名称 "u" 具有二义性
Invoke-WebRequest
```

**原因**

PowerShell 中 `curl` 是 `Invoke-WebRequest` 的别名。

**解决**

使用真正的 curl：

```
curl.exe
```

例如：

```
curl.exe -k -u elastic https://localhost:9200
```

------

### 5. Elasticsearch 认证失败

**报错**

```
security_exception
unable to authenticate user [elastic]
```

**原因**

输入密码错误。

**解决**

重置密码：

```
bin\elasticsearch-reset-password.bat -u elastic
```

------

### 6. IK 插件版本不匹配

**报错**

```
plugin [analysis-ik] was built for Elasticsearch version 9.x
but version 8.10.0 is running
```

**原因**

插件版本必须和 Elasticsearch **主版本完全一致**。

**解决**

安装匹配版本：

```
bin\elasticsearch-plugin.bat install https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-8.10.0.zip
```

------

### 7. 插件安装验证

```
curl.exe -k -u elastic:密码 https://localhost:9200/_cat/plugins?v
```

输出：

```
analysis-ik 8.10.0
```

说明插件安装成功。

------

## 最终状态

当前环境：

- OS：Windows 11
- Elasticsearch：8.10.0
- Security：已启用
- IK 分词器：已安装
- 访问方式：HTTPS + Basic Auth

测试：

```
https://localhost:9200
```

服务正常。

------

## 总结

主要踩坑点：

1. 旧版 `CLASSPATH` 环境变量
2. Windows / WSL 环境混用
3. PowerShell `curl` 别名
4. Elasticsearch 安全认证
5. 插件版本匹配

全部解决后即可正常使用 Elasticsearch 8.10。