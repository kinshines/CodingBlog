# GitHub生成SSH Key后提交更改

## 创建SSH Key
在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

>$ ssh-keygen -t rsa -C "youremail@example.com"

如果一切顺利的话，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

## 添加 SSH Key
然后，登陆GitHub，打开“Account settings”，“SSH Keys”页面：
然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容。点完成，得到ssh。

## 修改url
打开repo目录下的.git/config，url是HTTPS形式。

>[remote “origin”]
>fetch = + refs/heads/*:refs/remotes/origin/*
>url = https://username@github.com/username/projectname.git

因为远程版本库的url是HTTPS，所以问题就出在这了，每次提交都很不方便，都要输入用户名和密码。

为了使用SSH公钥的方式认证，把config的url改成下面这样

>[remote “origin”]
>fetch = + refs/heads/:refs/remotes/origin/
>url = git@github.com:username/projectname.git

这样，push时候就不用填帐号密码了。

