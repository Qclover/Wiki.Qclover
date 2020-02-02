### jekins未授权访问漏洞

未授权访问管理控制台,可以通过脚本命令行执行系统命令。 

**CVE-2019-1003000（jekins远程代码执行）**

Script Security and Pipeline插件是Jenkins的一个安全插件，可以集成到Jenkins各种功能插件中。 

jenkins 2.137版本搭建之后，不需要任何权限即可利用该漏洞执行任意代码达到命令执行的目的。从2.138版本开始，需要普通用户权限才可以利用该漏洞。 

Jenkins提供了一个界面给使用者检查自己的Pipeline脚本，可以在脚本编译阶段绕过脚本安全沙箱保护，导致具有Overall/Read权限或能控制SCM中jenkinsfile或沙盒Pipeline共享库内容的用户绕过沙盒保护，并在Jenkins服务器上执行任意代码。 