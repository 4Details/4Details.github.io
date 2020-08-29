---
title: 找不到请求的.Net Framework Data Provider.可能没有安装.  错误
date: 2019-07-27 16:23:27
tags: [.Net]
category: [编程笔记]
---

# 问题：

这几天在装.NET 的开发环境，在装好VS2013和Oracle 11g之后，做了一个测试项目，运行调试没问题
但是涉及到数据库相关操作，如新建数据集、连接数据库等在调试的时候则会出现如下错误：

![找不到请求的 .Net Framework Data Provider。可能没有安装](https://img-blog.csdn.net/20180724122125269?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIzODI0MjU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



# 目前百度之后现存的解决方案：

1. oracle odp.net 32位/64位版本的问题
   [解决方案链接](http://www.cnblogs.com/yjmyzz/archive/2011/04/19/2020793.html)；当然如果觉得这篇写的不怎么清楚，还可以点击 [这里](https://www.cnblogs.com/gudi/p/6110875.html)
   **\*（我碰到的就是这个问题，但是据博主提供的方法没有解决问题）***

2. Microsoft SQL Server Compact 4.0没有安装 这个问题是比较好解决的，只用安装Microsoft SQL Server Compact
   4.0即可，具体可以点击[这里](https://blog.csdn.net/yuchou123456789/article/details/7031206)

3. 还有修改machine.config配置文件的方法，大家也可以尝试[点击这里](http://qihuayu2010.blog.163.com/blog/static/18790015920138235546382/)

   当然如果你的数据库使用的Oracle，节点配置的时候需要根据实际情况做出调整。
   具体文件配置路径：C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config下面的machine.config
   用记事本打开，查看下列节点是否存在oracle的相关配置

   ```
   <system.data>
       <DbProviderFactories>
         <add name="ODP.NET, Managed Driver" invariant="Oracle.ManagedDataAccess.Client" description="Oracle Data Provider for .NET, Managed Driver" type="Oracle.ManagedDataAccess.Client.OracleClientFactory, Oracle.ManagedDataAccess, Version=4.121.2.0, Culture=neutral, PublicKeyToken=89b483f429c47342" />
         <add name="Microsoft SQL Server Compact Data Provider 4.0" invariant="System.Data.SqlServerCe.4.0" description=".NET Framework Data Provider for Microsoft SQL Server Compact" type="System.Data.SqlServerCe.SqlCeProviderFactory, System.Data.SqlServerCe, Version=4.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91" />
       </DbProviderFactories>
     </system.data>
   ```

   新添加的应该是这一块：

   ```
   <add name="ODP.NET, Managed Driver" invariant="Oracle.ManagedDataAccess.Client" description="Oracle Data Provider for .NET, Managed Driver" type="Oracle.ManagedDataAccess.Client.OracleClientFactory, Oracle.ManagedDataAccess, Version=4.121.2.0, Culture=neutral, PublicKeyToken=89b483f429c47342" />
   ```

   # 个人解决方案

   前面说过了，我尝试了上述的一些办法之后仍然没有解决问题，熬不住了我就去csdn的论坛发了帖，等了半个小时没人回复（可能是积分太少吧），无奈我就自己继续鼓捣了。想起来自己有一个 ODAC 12c的安装包，就直接点击安装了，安装完成之后重启VS，继续新建项目，配置数据库，调试之后竟然没有再继续报错，也就是说这个问题被我糊里糊涂解决了，哈哈哈~

   写一篇记录一下，给各位一个借鉴也给自己一个教训。
   下面提供ODAC 12c的下载地址

   官方下载地址：[x64下载](http://www.oracle.com/technetwork/database/windows/downloads/index-090165.html)、[x86下载](http://www.oracle.com/technetwork/topics/dotnet/utilsoft-086879.html)

   如果没有oracle账号又或是账号无法登陆，可以通过这个[链接](https://pan.baidu.com/s/13JEqOxtnqwVIu1ohEKds2g)下载， 密码：amvz