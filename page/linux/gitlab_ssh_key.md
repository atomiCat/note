# 记一次 gitlab-ce 添加ssh key无效问题
在服务器**code.xxx.com**（centos7）安装完gitlab-ce,注册登录创建仓库一切正常，http协议进行git操作正常。添加完ssh key后，
执行 **git clone** 
出现 
```bash
git@code.xxx.com's password:
```
## 解决方案
参考[stackoverflow](https://stackoverflow.com/questions/20440096/google-cloud-engine-permission-denied-publickey-gssapi-keyex-gssapi-with-mic/26204443#26204443)   
` restorecon -Rv /var/opt/gitlab/.ssh/ `  
或  
` semanage fcontext -a -t ssh_home_t "/var/opt/gitlab/.ssh/authorized_keys" `
### 解决过程

首先怀疑是gitlab创建的git账户有问题，于是先测试基本的ssh公私钥验证登录功能。  
先重置git@code.xxx.com的登录密码，然后将**另一台linux**的publicKey添加到git@code.xxx.com中：  
` ssh-copy-id -i id_rsa.pub git@code.xxx.com `  
然后 ` ssh git@code.xxx.com `  发现仍然要求输入密码  
下面以debug模式,222端口开启sshd服务，以便查看日志  
` /usr/sbin/sshd -p 222 -d `  
**另一台linux**连接222端口并打印详细日志  
`ssh -vvv -p 222 git@code.xxx.com`  
发现登录成功  
另外测试发现debug模式启动的sshd后，**git clone**也可以成功了  
修改sshd日志级别  
`vi /etc/ssh/sshd_config`  
找到**LogLevel**，将后面的INFO改为DEBUG  
保存重启sshd `systemctl restart sshd`  
查看sshd日志 `tail -f /var/log/secure`  
**另一台linux** ` ssh git@code.xxx.com `   
发现sshd日志出现  
` debug1: Could not open authorized keys '/var/opt/gitlab/.ssh/authorized_keys': Permission denied `  
然后复制并打开[stackoverflow](https://stackoverflow.com)粘贴回车一气呵成问题解决
