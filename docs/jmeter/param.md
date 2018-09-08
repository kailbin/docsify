# 命令行参数

```bash
$ jmeter -?
    _    ____   _    ____ _   _ _____       _ __  __ _____ _____ _____ ____
   / \  |  _ \ / \  / ___| | | | ____|     | |  \/  | ____|_   _| ____|  _ \
  / _ \ | |_) / _ \| |   | |_| |  _|    _  | | |\/| |  _|   | | |  _| | |_) |
 / ___ \|  __/ ___ \ |___|  _  | |___  | |_| | |  | | |___  | | | |___|  _ <
/_/   \_\_| /_/   \_\____|_| |_|_____|  \___/|_|  |_|_____| |_| |_____|_| \_\ 4.0 r1823414

Copyright (c) 1999-2018 The Apache Software Foundation

	# 命令行选项
	--?
		print command line options and exit
	########## 常见用法
	-h, --help
		print usage information and exit
	# 打印版本信息
	-v, --version
		print the version information and exit
	# 默认是使用JMETER_HOME/bin目录下的jmeter.properties
	-p, --propfile <argument>    
		the jmeter property file to use
	# 其它配置文件，如JVM参数等等
	-q, --addprop <argument>     
		additional JMeter property file(s)
	########## 指定测试计划文件(.jmx)
	-t, --testfile <argument>
		the jmeter test(.jmx) file to run. "-t LAST" will load last
		used file
	########## 指定生成测试结果保存的文件，jtl文件格式
	-l, --logfile <argument>
		the file to log samples to
	# 
	-i, --jmeterlogconf <argument>
		jmeter logging configuration file (log4j2.xml)
	########## 指定记录jmeter log的文件，默认为jmeter.log
	-j, --jmeterlogfile <argument>
		jmeter run log file (jmeter.log)
	########## 非GUI模式执行JMeter
	-n, --nongui
		run JMeter in nongui mode
	########## 运行JMeter server
	-s, --server
		run the JMeter server
		
	# 代理地址
	-H, --proxyHost <argument>
		Set a proxy server for JMeter to use
	# 代理端口
	-P, --proxyPort <argument>
		Set proxy server port for JMeter to use
	# 指定不进行代理的地址列表
	-N, --nonProxyHosts <argument>
		Set nonproxy host list (e.g. *.apache.org|localhost)
	# 代理用户名
	-u, --username <argument>
		Set username for proxy server that JMeter is to use
	# 代理密码
	-a, --password <argument>
		Set password for proxy server that JMeter is to use
		
	########## 指定JMeter 属性，只通过 ${__P(argument)} 获取
	-J, --jmeterproperty <argument>=<value>
		Define additional JMeter properties
	########## 定义全局属性，分布式压测压测下 会发送到各个机器
	-G, --globalproperty <argument>=<value>
		Define Global properties (sent to servers)
		e.g. -Gport=123
		 or -Gglobal.properties
		 
	# 指定系统属性
	-D, --systemproperty <argument>=<value>
		Define additional system properties
	# 指定系统属性文件
	-S, --systemPropertyFile <argument>
		additional system property file(s)
		
	##########在开始之前强制删除存在的测试结果文件
	-f, --forceDeleteResultFile
		force delete existing results files before start the test
	# 指定日志等级
	-L, --loglevel <argument>=<value>
		[category=]level e.g. jorphan=INFO, jmeter.util=DEBUG or com
		.example.foo=WARN
		
	########## 启动远程server（在jmeter property中定义好的remote_hosts），仅在non-gui模式生效
	-r, --runremote
		Start remote servers (as defined in remote_hosts)
	########## 启动远程server（如果使用此参数，将会忽略jmeter property中定义的remote_hosts）
	-R, --remotestart <argument>
		Start these remote servers (overrides remote_hosts)

	# 指定 JMeter home
	-d, --homedir <argument>
		the jmeter home directory to use
	# 测试结束时，退出远程 Server，仅在non-gui模式生效
	-X, --remoteexit
		Exit the remote servers at end of test (non-GUI)
	# 保存测试结果到 csv 文件
	-g, --reportonly <argument>
		generate report dashboard only, from a test results file
	########## 测试结束后，生成测试报告
	-e, --reportatendofloadtests
		generate report dashboard after load test
	########## 指定测试报告的存放位置
	-o, --reportoutputfolder <argument>
		output folder for report dashboard

```



# 使用示例

```bash
$ jmeter -h
    _    ____   _    ____ _   _ _____       _ __  __ _____ _____ _____ ____
   / \  |  _ \ / \  / ___| | | | ____|     | |  \/  | ____|_   _| ____|  _ \
  / _ \ | |_) / _ \| |   | |_| |  _|    _  | | |\/| |  _|   | | |  _| | |_) |
 / ___ \|  __/ ___ \ |___|  _  | |___  | |_| | |  | | |___  | | | |___|  _ <
/_/   \_\_| /_/   \_\____|_| |_|_____|  \___/|_|  |_|_____| |_| |_____|_| \_\ 4.0 r1823414

Copyright (c) 1999-2018 The Apache Software Foundation


To list all command line options, open a command prompt and type:

jmeter.bat(Windows)/jmeter.sh(Linux) -?

--------------------------------------------------

To run Apache JMeter in GUI mode, open a command prompt and type:

jmeter.bat(Windows)/jmeter.sh(Linux) [-p property-file]

--------------------------------------------------

To run Apache JMeter in NON_GUI mode:
Open a command prompt (or Unix shell) and type:

jmeter.bat(Windows)/jmeter.sh(Linux) -n -t test-file [-p property-file] [-l results-file] [-j log-file]

--------------------------------------------------

To run Apache JMeter in NON_GUI mode and generate a report at end :
Open a command prompt (or Unix shell) and type:

jmeter.bat(Windows)/jmeter.sh(Linux) -n -t test-file [-p property-file] [-l results-file] [-j log-file] -e -o [Path to output folder]

--------------------------------------------------
To generate a Report from existing CSV file:
Open a command prompt (or Unix shell) and type:

jmeter.bat(Windows)/jmeter.sh(Linux) -g [csv results file] -o [path to output folder (empty or not existing)]

--------------------------------------------------

To tell Apache JMeter to use a proxy server:
Open a command prompt and type:

jmeter.bat(Windows)/jmeter.sh(Linux) -H [your.proxy.server] -P [your proxy server port]

---------------------------------------------------

To run Apache JMeter in server mode:
Open a command prompt and type:

jmeter-server.bat(Windows)/jmeter-server(Linux)

---------------------------------------------------
```

