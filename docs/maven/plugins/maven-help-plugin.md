
# maven-help-plugin

Maven Help插件 可以用来查看构建目标的帮助文档。

## help:help

``` text
$ mvn help:help

The Maven Help plugin provides goals aimed at helping to make sense out of the
build environment. It includes the ability to view the effective POM and
settings files, after inheritance and active profiles have been applied, as
well as a describe a particular plugin goal to give usage information.

This plugin has 9 goals:

help:active-profiles
  Displays a list of the profiles which are currently active for this build.

help:all-profiles
  Displays a list of available profiles under the current project.
  Note: it will list all profiles for a project. If a profile comes up with a
  status inactive then there might be a need to set profile activation
  switches/property.

help:describe
  Displays a list of the attributes for a Maven Plugin and/or goals (aka Mojo -
  Maven plain Old Java Object).

help:effective-pom
  Displays the effective POM as an XML for this build, with the active profiles
  factored in.

help:effective-settings
  Displays the calculated settings as XML for this project, given any profile
  enhancement and the inheritance of the global settings into the user-level
  settings.

help:evaluate
  Evaluates Maven expressions given by the user in an interactive mode.

help:expressions
  Displays the supported Plugin expressions used by Maven.

help:help
  Display help information on maven-help-plugin.
  Call mvn help:help -Ddetail=true -Dgoal=<goal-name> to display parameter
  details.

help:system
  Displays a list of the platform details like system properties and environment
  variables.
```

显示目标的详细参数信息

```text
$ mvn help:help -Ddetail=true -Dgoal=<goal-name>
```

## help:describe


### 显示Maven插件的目标列表

```text
# 如果是官方插件，可以执行用 缩略名
mvn help:describe -Dplugin=dependency

# 也可以用完全限定名
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-dependency-plugin
# 或
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-dependency-plugin:3.0.2
```

### 显示详细信息
```text
mvn help:describe -Dplugin=dependency -Ddetail=true
```


## 显示某个目标的详细属性信息：
```text
mvn help:describe -Dplugin=dependency -Ddetail=true -Dgoal=tree
```


## help:active-profiles

显示可用的 profiles


## help:all-profiles

显示所有可用的 profiles 列表。如果某个profile 文件的状态为非活动状态，则可能需要设置profile文件激活开关/属性。



## help:effective-pom

显示最终生效的pom xml，并将激活的 profile 也考虑在内。  



## help:effective-settings

最终生效的 settings 文件设置



## help:evaluate


以交互模式评估用户给出的Maven表达式。




## help:expressions

显示Maven使用的受支持的插件表达式。


## help:system

显示系统属性和环境变量等平台详细信息列表。

