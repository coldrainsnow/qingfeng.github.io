---
layout: post
title: "AWS-SDK-CPP遇到的问题"
date: 2023-06-13
excerpt: "构造函数出错"
tags: [cpp]
comments: true
---

# 1.问题

程序在启动加载时非常慢，后来排查到时这行代码有问题Aws::S3::S3Client client

# 2.解决办法

在查AWS-SDK-CPP的仓库时，发现也有人提到了类似的问题，可以加个环境变量

先说最后可以做到的解决办法

在代码中添加环境变量或者在我的电脑高级设置中添加

```cpp
AWS_EC2_METADATA_DISABLED=true
```

其中在代码中添加时，需要把这个环境变量注册到path中，自己写一个函数

至于原因的话，是构造函数里面的问题，涉及的可能比较多，先不讨论了

# 3.官方回答

Sorry we are working on a better way to do it but currently the only way to avoid these involve environment variables. If the AWS_EC2_METADATA_DISABLED is problematic for you, you could also set AWS_DEFAULT_REGION or AWS_REGION to your region...
Though honestly if having the env variable as true causes troubles later on, the best workaround I can think of is having something like:

```
setenv("AWS_EC2_METADATA_DISABLED", "true", 1);
Aws::Client::ClientConfiguration clientConfig("default"); // use this configuration for all aws clients or do the same for other client configurations
setenv("AWS_EC2_METADATA_DISABLED", "false", 1);
```

either that or changing the source code directly- on clientConfiguration.cpp remove lines 127-134.
Hope one of these works for you.
Ah and also, we have this documented as part of the [changes on version 1.8 ](https://github.com/aws/aws-sdk-cpp/wiki/What’s-New-in-AWS-SDK-for-CPP-Version-1.8#client-configuration-now-reads-environment-variables-configuration-file-and-ec2-metadata-to-get-default-aws-region)in our wiki

简单说，要不加上这几行代码，要不就把clientConfiguration.cpp的127-134行代码删除

注意setenv是Linux添加环境变量的方法，windows是这样的

```cpp
SetEnvironmentVariable("AWS_EC2_METADATA_DISABLED", "true");
```

但我用windows的方法没用，不知道为什么

另外我在电脑里clientConfiguration.cpp也没有找到

所以最简单的方法，直接在电脑环境变量加上就好

# 4.参考

[c++ - Aws::S3::S3Client constructor very slow - Stack Overflow](https://stackoverflow.com/questions/63725308/awss3s3client-constructor-very-slow)

[Performance Degradation because of EC2 Metadata Client · Issue #1511 · aws/aws-sdk-cpp (github.com)](https://github.com/aws/aws-sdk-cpp/issues/1511)

[Severe Performance Degradation for S3 using SDK 1.8.32 · Issue #1440 · aws/aws-sdk-cpp (github.com)](https://github.com/aws/aws-sdk-cpp/issues/1440)





# 