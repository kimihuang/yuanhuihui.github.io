---
layout: post
title:  "SELinux SEAndroid权限问题"
date:   2015-04-10 22:10:54
categories: Android
excerpt:  系统运行过程中，权限不足，往往需要SELinux来解决
---

* content
{:toc}


---

## 权限修改

### 方法1： adb在线修改seLinux
 `Enforcing`(表示已打开)，`Permissive`（表示已关闭）

		getenforce;  //获取当前seLinux状态
		setenforce 1;   //打开seLinux
		setenforce 0;   //关闭seLinux

### 方法2：从kernel中彻底关闭
修改`LINUX/android/kernel/arch/arm64/configs/xxx_defconfig`文件（xxx一般为手机产品名）， 去掉`CONFIG_SECURITY_SELINUX=y `的配置项

### 方法3.sepolicy中添加权限

-  修改依据，通过指令`cat /proc/kmsg | grep denied`，或者kernel的Log中定位到标志性log。
    
- 修改步骤
	- 找相应的{源类型.te }文件，此文件可能的存放路径 (其中**源类型**见下方的标志性log格式) ：

			LINUX/android/external/sepolicy 
			LINUX/android/device/qcom/sepolicy/common

	- 标志性log 格式
	
			avc: denied  { 操作权限  }  for pid=7201  comm=“进程名”  scontext=u:r:源类型:s0  tcontext=u:r:目标类型:s0  tclass=访问类型 permissive=0

	- 在相应.te文件，添加如下格式的一行语句：(结尾别忘了分号)
	
			格式：allow  源类型 目标类型:访问类型 {权限}; 

- 实例

	- Kernel Log
	
			avc: denied {getattr read} for pid=7201 comm="xxx.xxx" scontext=u:r:system_app:s0 tcontext=u:r:shell_data_file:s0 tclass=dir permissive=0

	- 修改方案
			
			system_app.te文件中，添加下面语句：
			allow system_app shell_data_file:dir{getattr read};