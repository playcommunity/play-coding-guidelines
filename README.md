# Play Framework 开发规范 v0.1

## 目标
该开发规范主要目的是制定一些列开发风格和约定，遵守这些规则一方面可以让新人更快上手，另一方也可以培养团队的默契程度，在提升生产效率的同时，也增强了项目的可维护性。

## 应用配置
### 涉密配置项
涉密配置项不应该直接写在配置文件中，而是应该从环境变量中读取。例如在`conf/application.conf`文件中，通过如下方式引用环境变量中的数据库配置：
```
mongodb.uri = ${?MONGO_URI}
```

### 开发配置项
开发人员在本地调试时经常需要自定义一些配置项的值，例如外部API服务地址，并且使用这些自定义值覆盖 `conf/application.conf` 的同名配置项。我们把这些可以让开发人员自定义的配置项称为`开发配置项`。`开发配置项`需要在一个独立的配置文件`conf/dev.conf`中定义，并且该文件为私有文件，每个开发人员人手一份，不纳入版本控制系统。在`conf/application.conf`底部通过`include`语句引入`开发配置项`：
```
include "dev"
```
然后开发人员就可以在`conf/dev.conf`文件中增加自定义配置项，例如自定义`mongodb.uri`配置：
```
mongodb.uri = "mongodb://user:password@host:port/db?authMode=scram-sha1"
```
最后将`conf/dev.conf`加入`.gitignore`文件，提示Git忽略该文件：
```
logs
target
...
conf/dev.conf
```

## MongoDB设计
### 实时同步设计
在利用 `ChangeStreams` 实现实时同步功能时，要注意以下设计要点：
- 如果 Collection 未设计删除字段标记，则尽量使用有意义的主键，例如用户表主键`_id`存放用户的唯一标识(如邮箱)。这样设计的目的是因为删除操作在 `ChangeStreams` 中仅包含`_id`信息而不包含源文档信息。
- Collection尽量包含方便记录日志字段，如`addTime`, `updateTime`, `operator`。

### 性能优化
- 如果 Collection 包含一个内容较长的字符串类型字段，并且需要经常执行`$eq`查询，则应该为该字段新增一个带索引的hash字段，在执行`$eq`查询时使用hash值进行比较。