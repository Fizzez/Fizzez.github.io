---
title: "SSH原理和Github多用户设置"
date: 2020-08-28 08:35:28
tags:
- System
- Git
- Chinese
---

# SSH原理

Secure Shell (SSH)，是一种加密的网络传输协议，最常见的用途是登录到远程电脑中执行命令。SSH使用客户端-服务器模型，标准端口为22。SSH以非对称加密实现身份验证（关于对称以及非对称加密， 详见参考[2]）。从客户端来看，SSH提供两种级别的安全验证：

1. **基于密码的安全验证**：使用自动生成的“公钥-私钥对”来简单的加密网络链接，随后使用密码认证进行登录。所有传输的数据都会被加密，但是可能会有别的服务器在冒充真正的服务器，无法避免被中间人攻击。
2. **基于密钥的安全验证**：创建”公钥-私钥对“（e.g. 使用`ssh-keygen`命令），通过生成的密钥进行认证。公钥放在需要访问的服务器上（通常在`~/.ssh/authorized_keys`文件中），而对应的私钥由客户端保管。客户端软件会向服务器发出请求，请求用你的密钥进行安全验证。服务器收到请求后，先在你在该服务器的用户根目录下寻找你的公钥，然后把它和你发过来的公钥进行比较。如果两者一致，服务器就用公钥加密“质询”（challenge）并把它发给客户端软件，从而避免被中间人攻击。

## 基于密码的安全验证流程

![基于密码的安全验证流程](/images/what-is-ssh/ssh_pwd_base.png)

1. Server收到Client的登录请求，并把自己的公钥发送给Client。
2. Client使用这个公钥将登陆密码进行加密。
3. Client将加密后的密码发送给Server。
4. Server使用自己的私钥将消息解密得到登陆密码，并验证其合法性。
5. 根据验证结果，给Client相应的响应。

## 基于密钥的安全验证流程

![](/images/what-is-ssh/ssh_pubkey_base.png)

1. Client侧生成一对公钥和私钥，并将自己的公钥存放在Server端，追加在文件`authorized_keys`中。
2. Server收到客户端的连接请求后，在`authorized_keys`中匹配到Client事先保存的的公钥`pubKey`，并生成随机数`R`，用Client的公钥对该随机数进行加密得到`PubKey(R)`，然后将加密后的信息发送给Client。
3. Client通过私钥对消息解密得到随机数`R`，然后对随机数和本次会话的`SessionKey`利用MD5生成摘要`Digest1`，发送给Server。
4. Server也会对同样的`R`和`SessionKey`利用同样的摘要算法生成`Digest2`。
5. Server端比较`Digest1`和`Digest2`是否相同，给Client相应响应。

# SSH实践：Github多用户设置（macOS）

## 生成密钥

`ssh-keygen`命令用于为ssh生成，管理和转换认证密钥。该命令有几个常用选项，`-t`用于指定加密方式，可选`dsa | ecdsa | ed25519 | rsa`， 一般为`rsa`；`-C`用于指定注释，通常使用自己的邮件名作为注释；`-b`用于制定密钥长度，对于RSA加密方式，最低长度为1024（可被破解），默认长度为2048，建议为2048或4096。示例如下：

```
$ ssh-keygen -t rsa -C "your_email@example.com" -b 2048
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa): /home/username/.ssh/GitHub_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/username/.ssh/GitHub_rsa.
Your public key has been saved in /home/username/.ssh/GitHub_rsa.pub.
The key fingerprint is:
SHA256:GcK7ORvFzH6fzA7qPmnzBr1DOWho5cCVgIpLkh6VGb8 Fan@outlook.com
The key's randomart image is:
+---[RSA 2048]----+
|   .+... .       |
|   +o.  o        |
| o.. oo..        |
|+o.   +*.o       |
|+..  E.=So .     |
|..    o== =      |
|     .=..+oo     |
|       +=o+= .   |
|      .++=.o*    |
+----[SHA256]-----+
```

公钥是一串很长的字符，为了便于肉眼比对和识别，所以有了指纹这东西；指纹位数短，更便于识别且与公钥一一对应。

指纹的用处之一是在使用SSH第一次连接到某主机时，会返回该主机使用的公钥的指纹让你识别。示例：

```
The authenticity of host 'hostname' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)?
```

## 在本机配置多个Github账户

使用以上命令生成两个ssh公钥-密钥对，保存为不同的名称，例如`github_user1_rsa`和`github_user2_rsa`。则会生成四个文件如下：

```
-rw-------    1 username  staff   1.8K Oct 28  2019 github_user1_rsa
-rw-r--r--    1 username  staff   407B Oct 28  2019 github_user1_rsa.pub
-rw-------    1 username  staff   1.8K Oct 28  2019 github_user2_rsa
-rw-r--r--    1 username  staff   409B Oct 28  2019 github_user2_rsa.pub
```

配置`~/.ssh/config`文件如下，如果没有该文件则自行创建一个：

```
# github user(your_email_1@example.com)
Host github_user1.github.com
HostName github.com
IdentityFile ~/.ssh/github_user1_rsa
User git

# github user2(your_email_2@example.jp)
Host github_user2.github.com
HostName github.com
IdentityFile ~/.ssh/github_user2_rsa
User git
```

其中`Host`字段为`HostName`的别名，用以区分`github_user1`和`github_user2`。`git`命令会用repo设置中的Hostname来匹配此处的`Host`别名，如果匹配成功则访问此处`Host`对应的`HostName`；

`IdentifyFile`制定了密钥文件的地址，注意是私钥；

`HostName`指定了要连接的服务器；

`User`指定了登录用户名，此处为`git`。

## 将SSH公钥添加到Github

分别在两个Github账户的Settings - SSH and GPG keys页面添加SSH key。如下图所示，Key字段需要复制粘贴对应的`github_user1_rsa.pub`或者`github_user2_rsa.pub`公钥文件中的内容。

![Add SSH key in Github](/images/what-is-ssh/github-ssh.png)

## 测试配置是否成功

使用以下命令测试配置是否成功：

```
# Address of github_user_1
ssh -T git@github_user_1.github.com 
# 出现如下内容，表示使用github_user_1的身份成功链接Github
Hi github_user_1! You've successfully authenticated, but GitHub does not provide shell access.

# Address of github_user_2
ssh -T git@github_user_2.github.com 
# 出现如下内容，表示使用github_user_2的身份成功链接Github
Hi github_user_2！ You've successfully authenticated, but GitHub does not provide shell access.
```

## 配置成功后的使用

以`git clone`为例，使用`github_user_1`的身份进行操作：

```
git clone git@github_user_1.github.com:fastai/fastai_dev.git
```

注意此处`git@`后的host名为`~/.ssh/config`中，为`github_user_1`指定的`Host`。

如果对已有repo修改push地址，则需要修改repo根目录下`.git/config`中`remote`字段内容：

```
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
        precomposeunicode = true
[remote "origin"]
        # 此处将github.com为Host别名
        url = git@github.com:fastai/fastai_dev.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
[user]
        name = github_user1
        email = your_email_1@example.com
```

因为在上面配置SSH的config文件时，用指定的Host别名替代了原来的Hostname，所以现在应该对不同用户使用不同的Host别名来替代Hostname。

另外，在项目根目录分别设置用户名和邮箱的方法是：

```
git config user.name github_user1
git config user.email your_email_1@example.com
```

# 参考

1. [Secure Shell - 'Wikipedia'](https://zh.wikipedia.org/zh-cn/Secure_Shell)
2. [图解SSH原理-'TopGun_Viper'](https://www.jianshu.com/p/33461b619d53)
3. [SSH key的介绍与在Git中的使用 - 'faner'](https://www.jianshu.com/p/1246cfdbe460)
4. [~/.ssh/config文件的使用 - '阳台的晾衣架'](https://www.jianshu.com/p/45201d18cc7c)
5. [Git多用户，不同项目配置不同Git账号 - 'Lange0x0'](https://blog.csdn.net/onTheRoadToMine/article/details/79029331)

