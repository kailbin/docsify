# Versions Maven Plugin

当 Maven 下面的子模块比较多的时候，每次修改工程的版本号都是一件非常的痛苦的事情，因为子模块都引用了顶级父模块的pom，所以虽然在父模块中定义了工程的版本号，但每个子模块中要显示地指定父模块的版本号，否则无法找到父模块的pom。

`versions-maven-plugin` 插件是一个可以自动修改项目版本号的工具，避免每次版本升级 都需要修改所有子模块的 parent 版本号。


## 快速开始

```bash
# 也可以使用全名称 
# mvn org.codehaus.mojo:versions-maven-plugin:set -DnewVersion=5.0.0-SNAPSHOT
$ mvn versions:set -DnewVersion=5.0.0-SNAPSHOT

# 执行成功后，所有子模块的 parent 版本已经修改，对原来的 pom 文件进行了备份
$ git status
#> Changes not staged for commit:
#>         modified:   example-client-simple/pom.xml
#>         modified:   example-client/pom.xml
#>         modified:   example-core/pom.xml
#>         modified:   example-server/pom.xml
#>         modified:   pom.xml
#> 
#> Untracked files:
#>         example-client-simple/pom.xml.versionsBackup
#>         example-client/pom.xml.versionsBackup
#>         example-core/pom.xml.versionsBackup
#>         example-server/pom.xml.versionsBackup
#>         pom.xml.versionsBackup

# 如果想要还原，可以执行
$ mvn versions:revert

# 如果确认要使用新版本，可以提交本次版本变更，这时会删除 .versionsBackup 备份文件
$ mvn versions:commit 

# 如果要升级成正式版，去掉 -SNAPSHOT 可重新执行 set 命令
# -DgenerateBackupPoms=false 不生成备份文件，免去 mvn versions:commit 的过程
# 可通过 git reset --hard HEAD 还原
$ mvn versions:set -DnewVersion=5.0.0 -DgenerateBackupPoms=false
```

## 查看插件帮助

```bash
$ mvn help:describe -Dplugin=versions
#> ...

#> [INFO] org.codehaus.mojo:versions-maven-plugin:2.7

#> Name: Versions Maven Plugin
#> Description: Versions Plugin for Maven. The Versions Plugin updates the
#> versions of components in the POM.
#> Group Id: org.codehaus.mojo
#> Artifact Id: versions-maven-plugin
#> Version: 2.7
#> Goal Prefix: versions

#> This plugin has 31 goals:

#> versions:commit
#> Description: Removes the initial backup of the pom, thereby accepting the
#>     changes.

#> versions:compare-dependencies
#> Description: Compare dependency versions of the current project to
#>     dependencies or dependency management of a remote repository project. Can
#>     optionally update locally the project instead of reporting the comparison

#> versions:dependency-updates-report
#> Description: Generates a report of available updates for the dependencies
#>     of a project.

#> versions:display-dependency-updates
#> Description: Displays all dependencies that have newer versions available.
#>     It will also display dependencies which are used by a plugin or defined in
#>     the plugin within a pluginManagement.

#> versions:display-parent-updates
#> Description: Displays any updates of the project's parent project

#> versions:display-plugin-updates
#> Description: Displays all plugins that have newer versions available,
#>     taking care of Maven version prerequisites.

#> versions:display-property-updates
#> Description: Displays properties that are linked to artifact versions and
#>     have updates available.

#> versions:force-releases
#> Description: Replaces any -SNAPSHOT versions with a release version, older
#>     if necessary (if there has been a release).

#> versions:help
#> Description: Display help information on versions-maven-plugin.
#>     Call mvn versions:help -Ddetail=true -Dgoal=<goal-name> to display
#>     parameter details.

#> versions:lock-snapshots
#> Description: Attempts to resolve unlocked snapshot dependency versions to
#>     the locked timestamp versions used in the build. For example, an unlocked
#>     snapshot version like '1.0-SNAPSHOT' could be resolved to
#>     '1.0-20090128.202731-1'. If a timestamped snapshot is not available, then
#>     the version will remained unchanged. This would be the case if the
#>     dependency is only available in the local repository and not in a remote
#>     snapshot repository.

#> versions:plugin-updates-report
#> Description: Generates a report of available updates for the plugins of a
#>     project.

#> versions:property-updates-report
#> Description: Generates a report of available updates for properties of a
#>     project which are linked to the dependencies and/or plugins of a project.

#> versions:resolve-ranges
#> Description: Attempts to resolve dependency version ranges to the specific
#>     version being used in the build. For example a version range of '[1.0,
#>     1.2)' would be resolved to the specific version currently in use '1.1'.

#> versions:revert
#> Description: Restores the pom from the initial backup.

#> versions:set
#> Description: Sets the current project's version and based on that change
#>     propagates that change onto any child modules as necessary.

#> versions:set-property
#> Description: Set a property to a given version without any sanity checks.
#>     Please be careful this can lead to changes which might not build anymore.
#>     The sanity checks are done by other goals like update-properties or
#>     update-property etc. they are not done here. So use this goal with care.

#> versions:set-scm-tag
#> Description: Updates the current project's SCM tag.

#> versions:unlock-snapshots
#> Description: Attempts to resolve unlocked snapshot dependency versions to
#>     the locked timestamp versions used in the build. For example, an unlocked
#>     snapshot version like '1.0-SNAPSHOT' could be resolved to
#>     '1.0-20090128.202731-1'. If a timestamped snapshot is not available, then
#>     the version will remained unchanged. This would be the case if the
#>     dependency is only available in the local repository and not in a remote
#>     snapshot repository.

#> versions:update-child-modules
#> Description: Scans the current projects child modules, updating the
#>     versions of any which use the current project to the version of the current
#>     project.

#> versions:update-parent
#> Description: Sets the parent version to the latest parent version.

#> versions:update-properties
#> Description: Sets properties to the latest versions of specific artifacts.

#> versions:update-property
#> Description: Sets a property to the latest version in a given range of
#>     associated artifacts.

#> versions:use-dep-version
#> Description: (no description available)

#> versions:use-latest-releases
#> Description: Replaces any release versions with the latest release version.

#> versions:use-latest-snapshots
#> Description: Replaces any release versions with the latest snapshot version
#>     (if it has been deployed).

#> versions:use-latest-versions
#> Description: Replaces any version with the latest version.

#> versions:use-next-releases
#> Description: Replaces any release versions with the next release version
#>     (if it has been released).

#> versions:use-next-snapshots
#> Description: Replaces any release versions with the next snapshot version
#>     (if it has been deployed).

#> versions:use-next-versions
#> Description: Replaces any version with the latest version.

#> versions:use-reactor
#> Description: Replaces any versions with the corresponding version from the
#>     reactor.

#> versions:use-releases
#> Description: Replaces any -SNAPSHOT versions with the corresponding release
#>     version (if it has been released).

#> For more information, run 'mvn help:describe [...] -Ddetail'
#> ...


# 打印出插件 所有 goal 的 所有参数使用帮助
$ mvn help:describe -Dplugin=versions -Ddetail

# 打印出指定的 goal 的详细帮助文档，包含每一个参数
$ mvn help:describe -Dplugin=versions -Ddetail -Dgoal=set
```

## 常用 goal 和 参数

| 命令                                                   | 描述                                              |
| ------------------------------------------------------ | ------------------------------------------------- |
| `mvn versions:set -DnextSnapshot=true`                 | 版本号自动加一，并且自动带上`-SNAPSHOT`           |
| `mvn versions:commit`                                  | 删除备份文件                                      |
| `mvn versions:set -DremoveSnapshot=true`               | 删除 `-SNAPSHOT` 改为正式版                       |
| `mvn versions:set -DnextSnapshot=true versions:commit` | 同时执行两个 goal，版本号升级后，自动删版备份文件 |
| `mvn versions:use-releases -Dincludes=*:check-core`    | 修改项目依赖的 `-SNAPSHOT` 为正式版               |

## 其他技巧

- 子模块中的 `<version>x.x.x</version>` 可以省略，默认与 parent 的版本号保持一致
- 子模块的版本号也可以写成 `<version>${parent.version}</version>` 与 parent 的版本号保持一致
- 子模块间互相依赖时，版本号可以 写成 `<version>${project.version}</version>` 或  `<version>${parent.version}</version>`  统一于 与 parent 的版本号保持一致

## Read  More

> 更多高级功能详见 [Versions Maven Plugin 官网](http://www.mojohaus.org/versions-maven-plugin/)



