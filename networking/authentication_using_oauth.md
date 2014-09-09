# 使用开放授权登陆验证（Authentication using OAuth）

OAuth是一个开放协议，允许简单的安全验证，是来自web的典型方法，用于移动和桌面应用程序。使用OAuth对通常的web服务的客户端进行身份验证，例如Google，Facebook和Twitter。

**注意**

**对于自定义的web服务，你也可以使用典型的HTTP身份验证，例如使用XMLHttpRequest的用户名和密码的获取方法（比如xhr.open(verb,url,true,username,password））。**

Auth目前不是QML/JS的接口，你需要写一些C++代码并且将身份验证导入到QML/JS中。另一个问题是安全的存储访问密码。

下面这些是我找到的有用的连接：

* http://oauth.net

* http://hueniverse.com/oauth/

* https://github.com/pipacs/o2

* http://www.johanpaul.com/blog/2011/05/oauth2-explained-with-qt-quick/
