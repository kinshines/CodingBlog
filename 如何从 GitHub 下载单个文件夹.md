# 如何从 GitHub 下载单个文件夹
GitHub上有的个别项目过大，但只需要下载其中一个文件夹，下面介绍使用SVN实现部分文件夹的下载。
### 将 /tree/main/ 换成 /trunk/ 
例如原URL：[dotnet/aspnetcore/tree/main/docs](https://github.com/dotnet/aspnetcore/tree/main/docs)

转换为URL：[dotnet/aspnetcore/trunk/docs](https://github.com/dotnet/aspnetcore/trunk/docs)

然后就可以使用 SVN 的 **Checkout** 签出这部分的代码

### 非 main 分支换成 /branches/branchname/
例如原URL：[dotnet/aspnetcore/tree/release/3.1/docs](https://github.com/dotnet/aspnetcore/tree/release/3.1/docs)

转换为URL：[dotnet/aspnetcore/branches/release/3.1/docs](https://github.com/dotnet/aspnetcore/branches/release/3.1/docs)

同样可以使用 SVN 的 **Checkout** 签出这个分支的代码

