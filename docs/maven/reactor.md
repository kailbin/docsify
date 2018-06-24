
# 反应堆（Reactor）

在多模块Maven项目中，**反应堆（Reactor）是一个包含了所有需要构建模块的抽象概念**。



在默认情况下，Maven会根据多模块配置构建所有的模块，Maven还会根据模块间的依赖关系自动计算构建顺序，以确保被依赖的模块会先得以构建。值得一提的是，在这种情形下，Maven会将父模块看成是其子模块的依赖。


 
一般来说，我们要么构建整个项目，要么构建单个模块，但是有些时候，我们会想要**仅仅构建这个完整的反应堆中的某些模块**，换句话说，我们会需要裁剪反应堆。

## 参数

Maven提供了很多命令行选项让我们自定义反应堆，输入`mvn -h`可以看到这样一些选项：

```
$ mvn -h

usage: mvn [options] [<goal(s)>] [<phase(s)>]

Options:
 -am,--also-make                        处理选定模块所依赖的模块
                                        
 -amd,--also-make-dependents            同时处理依赖选定模块的模块
                                        
 -B,--batch-mode                        Run in non-interactive (batch) mode
                                        
 -b,--builder <arg>                     The id of the build strategy to use.
                                        
 -C,--strict-checksums                  Fail the build if checksums don't match
                                        
 -c,--lax-checksums                     Warn if checksums don't match
 
 -cpu,--check-plugin-updates            Ineffective, only kept for backward compatibility
                                        
 -D,--define <arg>                      定义系统属性
 
 -e,--errors                            Produce execution error messages
 
 -emp,--encrypt-master-password <arg>   Encrypt master security password
 
 -ep,--encrypt-password <arg>           Encrypt server password
 
 -f,--file <arg>                        指定POM文件（或者pom.xml的目录）。
                                        
 -fae,--fail-at-end                     Only fail the build afterwards; allow all non-impacted builds to continue
                                        
 -ff,--fail-fast                        Stop at first failure in reactorized builds
                                        
 -fn,--fail-never                       NEVER fail the build, regardless of project result
                                        
 -gs,--global-settings <arg>            指定全局 setting 文件
                                        
 -gt,--global-toolchains <arg>          Alternate path for the global toolchains file
                                        
 -h,--help                              查看帮助信息
 
 -l,--log-file <arg>                    Log file where all build output will go.
                                        
 -llr,--legacy-local-repository         Use Maven 2 Legacy Local Repository behaviour, ie no use of _remote.repositories. 
                                        Can also be activated by using -Dmaven.legacyLocalRepo=true
                                        
 -N,--non-recursive                     表示不递归子模块
 
 -npr,--no-plugin-registry              废弃
                                        
 -npu,--no-plugin-updates               废弃
                                        
 -nsu,--no-snapshot-updates             禁止 SNAPSHOT 更新
 
 -o,--offline                           Work offline
 
 -P,--activate-profiles <arg>           激活profiles，多个用逗号分割
                                        
 -pl,--projects <arg>                   用于构建指定项目而不是所有项目，多个项目用逗号分隔。 项目可以由 '[groupId]：artifactId' 或其 '相对路径' 指定。A 
                                        
 -q,--quiet                             静默输出，只显示错误信息
 
 -rf,--resume-from <arg>                表示从指定模块开始继续处理
                                        
 -s,--settings <arg>                    指定 setting 文件
                                        
 -T,--threads <arg>                     Thread count, for instance 2.0C where C is core multiplied
                                        
 -t,--toolchains <arg>                  Alternate path for the user toolchains file
                                        
 -U,--update-snapshots                  强制更新快照依赖
                                        
 -up,--update-plugins                   Ineffective, only kept for backward compatibility
                                        
 -V,--show-version                      显示版本信息，并构建
                                        
 -v,--version                           显示版本信息
 
 -X,--debug                             显示调试信息
```

## 反应堆相关参数

- `-pl	--projects`	 选项后可跟随`[{groupId}:]{artifactId}`或者`所选模块的相对路径`(多个模块以逗号分隔)
- `-am	--also-make` 表示同时处理选定模块所依赖的模块
- `-amd	--also-make-dependents`	表示同时处理依赖选定模块的模块
- `-N	--Non-recursive` 表示不递归子模块
- `-rf	--resume-from`	表示从指定模块开始继续处理




## Read More

- [Maven的-pl -am -amd参数学习](https://www.cnblogs.com/hiver/p/7850954.html)
- [按需构建多模块，玩转Maven反应堆](http://juvenshun.iteye.com/blog/565240)
