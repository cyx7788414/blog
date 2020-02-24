# 背景
近日整理卡比窝租在万网（现阿里云）的共享空间，重新规划了资源目录，计划在原有的域名下根据目录对功能进行拆分，以备后续内容扩充，其中一个目录用于承载卡比窝的前端页面，因此需要对站点的进行转发配置。  
经过查阅资料，发现共享空间启用了Apache服务器的.htaccess配置文件，可以在每个目录下增加名为.htaccess的文件来对请求进行处理，因为原始文档是英文的，网上的经验帖又比较晦涩，因此总结一些没有注意到的细节以备后续查阅。
# 一个可以降低复杂度的原则
刚开始设计.htaccess规则的时候，总是想着一步到位，一个.htaccess文件搞定，从根目录直接把各个子目录的规则都配置到位。  
但在不熟练的情况下，复杂的设计会把思路搅成一团乱麻。  
这个时候，要记得一点，.htaccess文件可以存在于任何一个目录下，在子目录中进行处理时，会自动将请求路径与本级目录相符的部分省去，减少处理匹配路径的麻烦。  
因此，在不考虑性能的前提下，对于新手而言，这个有效降低复杂度的原则就是——**让每一级目录只做自己这一级的事**  
# 准备工作  
准备一个空白文件，起一个除了.htaccess以外的名字，因为FTP直接往共享空间里传是传不上去的，但是可以传上去了以后再改名字~  
空白文件里可以填上如下内容  
```.htaccess
<IfModule mod_rewrite.c>
Options +FollowSymlinks  #出于玄学目的加上的一句，暂时没有不良影响
RewriteEngine On 
RewriteBase /  #只在根目录的.htaccess文件里使用

#在这两条注释的位置准备填入配置项

</IfModule>
```
注意这里的文件头尾已经填好了，大意是开启Rewrite模块，网上相关资料大同小异，但是这里的原理可能和Apache服务器配置有关联，此处不作深究。 
# 条件与规则
转发配置的核心只有两样，代表条件的RewriteCond与代表规则的RewriteRule，可以简单理解成如下伪代码：
```javascript
if (RewriteCond_1) {//跳转条件
    RewriteRule_1 //跳转规则
    return;
}
if (RewriteCond_2) {//跳转条件
    RewriteRule_2 //跳转规则
    return;
}
...
```
当然，上述伪代码里的RewriteCond与RewriteRule都可能是复数条目的集合，这里面的逻辑在下面详解，而if语句里的return想说明的一点是，如果已经判断进入了某个RewriteCond集合，那么是不会再进行后续RewriteCond集合的匹配。  
## 先来一颗糖
在一些能看到的一些实例里，可以看见以%{}包裹起来的字段，例如%{DOCUMENT_ROOT}，这里是PHP提供的$_SERVER预定义服务器变量，可以反馈一些服务器与请求相关的信息用于条件与规则匹配执行，具体内容如下：
```php
$_SERVER['PHP_SELF'] #当前正在执行脚本的文件名，与 document root相关。
$_SERVER['argv'] #传递给该脚本的参数。
$_SERVER['argc'] #包含传递给程序的命令行参数的个数（如果运行在命令行模式）。
$_SERVER['GATEWAY_INTERFACE'] #服务器使用的 CGI 规范的版本。例如，“CGI/1.1”。
$_SERVER['SERVER_NAME'] #当前运行脚本所在服务器主机的名称。
$_SERVER['SERVER_SOFTWARE'] #服务器标识的字串，在响应请求时的头部中给出。
$_SERVER['SERVER_PROTOCOL'] #请求页面时通信协议的名称和版本。例如，“HTTP/1.0”。
$_SERVER['REQUEST_METHOD'] #访问页面时的请求方法。例如：“GET”、“HEAD”，“POST”，“PUT”。
$_SERVER['QUERY_STRING'] #查询(query)的字符串。
$_SERVER['DOCUMENT_ROOT'] #当前运行脚本所在的文档根目录。在服务器配置文件中定义。
$_SERVER['HTTP_ACCEPT'] #当前请求的 Accept: 头部的内容。
$_SERVER['HTTP_ACCEPT_CHARSET'] #当前请求的 Accept-Charset: 头部的内容。例如：“iso-8859-1,*,utf-8”。
$_SERVER['HTTP_ACCEPT_ENCODING'] #当前请求的 Accept-Encoding: 头部的内容。例如：“gzip”。
$_SERVER['HTTP_ACCEPT_LANGUAGE']#当前请求的 Accept-Language: 头部的内容。例如：“en”。
$_SERVER['HTTP_CONNECTION'] #当前请求的 Connection: 头部的内容。例如：“Keep-Alive”。
$_SERVER['HTTP_HOST'] #当前请求的 Host: 头部的内容。
$_SERVER['HTTP_REFERER'] #链接到当前页面的前一页面的 URL 地址。
$_SERVER['HTTP_USER_AGENT'] #当前请求的 User-Agent: 头部的内容。
$_SERVER['HTTPS'] #— 如果通过https访问,则被设为一个非空的值(on)，否则返回off
$_SERVER['REMOTE_ADDR'] #正在浏览当前页面用户的 IP 地址。
$_SERVER['REMOTE_HOST'] #正在浏览当前页面用户的主机名。
$_SERVER['REMOTE_PORT'] #用户连接到服务器时所使用的端口。
$_SERVER['SCRIPT_FILENAME'] #当前执行脚本的绝对路径名。
$_SERVER['SERVER_ADMIN'] #管理员信息
$_SERVER['SERVER_PORT'] #服务器所使用的端口
$_SERVER['SERVER_SIGNATURE'] #包含服务器版本和虚拟主机名的字符串。
$_SERVER['PATH_TRANSLATED'] #当前脚本所在文件系统（不是文档根目录）的基本路径。
$_SERVER['SCRIPT_NAME'] #包含当前脚本的路径。这在页面需要指向自己时非常有用。
$_SERVER['REQUEST_URI'] #访问此页面所需的 URI。例如，“/index.html”。
$_SERVER['PHP_AUTH_USER'] #当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的用户名。
$_SERVER['PHP_AUTH_PW'] #当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的密码。
$_SERVER['REQUEST_TIME'] #中保存了发起该请求时刻的时间戳, 此参数在 PHP 5.1及以后版本中可用
```
## 条件
RewriteCond的语法规则如下：
```
RewriteCond TestString CondPattern [flags]
```
其大意便是用TestString来与CondPatten进行匹配，如果匹配成功，则此条RewriteCond通过，而flags对其进行修饰限定。  
其中，TestString是一个字符串，除了普通字符，还可以包含前面说到的服务器变量，在资料中还提到可以包含后向引用与RepeatMap扩展，受限于应用水平，此处不作深究。  
CondPatten可以理解成一个正则表达式，但很明显的区别是由于标准正则的ig等参数在此处被放在了flags字段当做，所以这里不再需要使用两条斜杠将正则表达式包裹，仅在有需要的时候以^代表开头，以$代表结尾。  
在某些特殊场景下，CondPattern中以>、<、=符号加上普通字符串可以进行字符串比较，以!开头表示非，在此之外还有一些特例如下：  
> '-d' (是否为目录) 将testString当作一个目录名，检查它是否存在以及是否是一个目录。  
> '-f' (是否是regular file) 将testString当作一个文件名，检查它是否存在以及是否是一个regular文件。  
> -s' (是否为长度不为0的regular文件) 将testString当作一个文件名，检查它是否存在以及是否是一个长度大于0的regular文件。  
> '-l' (是否为symbolic link) 将testString当作一个文件名，检查它是否存在以及是否是一个 symbolic link。  
> '-F' (通过subrequest来检查某文件是否可访问) 检查TestString是否是一个合法的文件，而且通过服务器范围内的当前设置的访问控制进行访问。这个检查是通过一个内部subrequest完成 的, 因此需要小心使用这个功能以降低服务器的性能。  
> '-U' (通过subrequest来检查某个URL是否存在) 检查TestString是否是一个合法的URL，而且通过服务器范围内的当前设置的访问控制进行访问。这个检查是通过一个内部subrequest完成 的, 因此需要小心使用这个功能以降低服务器的性能。  
>
而flags字段由于与RewriteRule中的flags字段通用，因此在后面单独说明。
## 规则
RewriteRule的语法规则如下：
```
RewriteRule Pattern Substitution [flags]
```
用以将当前文件夹下的请求URL与Pattern进行匹配，将匹配到的内容用Substitution进行替换。  
其中Pattern是与RewriteCond中的CondPattern高度类似的正则表达式，但此处一般用于匹配后的替换，所以往往进行分组匹配以帮助进行参数提取复用。  
而匹配成功后，Substitution用来进行替换，其中除了普通字符，还可以使用$N来引用Pattern中的分组匹配结果，使用%N来引用最后一个RewriteCond中的匹配结果， 还可以使用前面提到的服务器变量与RewriteMap映射函数。  
这里有个特例，当SubStitution以-开头时，只检查不替换，因此可以实现不进行处理直接返回的效果。  
而flags字段由于与RewriteCond中的flags字段通用，因此在后面单独说明。
## 标记
flags可以理解成是一个字符串数组，即以[]包裹，以逗号隔开的一个或一组字符，其中字符有特定的含义，如下列表格式为(```'说明|对应字符' 解释```)：  
> 'nocase|NC'(no case)忽略大小  
> 'ornext|OR' (or next condition)逻辑或，可以同时匹配多个RewriteCond条件RewriteRule适用的标志符  
> 'redirect|R [=code]' (force redirect)强迫重写为基于http开头的外部转向(注意URL的变化) 如：[R=301,L]  
> 'forbidden|F' (force URL to be forbidden)重写为禁止访问  
> 'proxy|P' (force proxy)重写为通过代理访问的http路径  
> 'last|L' (last rule)最后的重写规则标志，如果匹配，不再执行以后的规则  
> 'next|N' (next round)循环同一个规则，直到不能满足匹配  
> 'chain|C' (chained with next rule)如果匹配该规则，则继续下面的有Chain标志的规则  
> 'type|T=MIME-type' (force MIME type)指定MIME类型  
> 'nosubreq|NS' (used only if no internal sub-request)如果是内部子请求则跳过  
> 'nocase|NC' (no case)忽略大小  
> 'qsappend|QSA' (query string append)附加查询字符串  
> 'noescape|NE' (no URI escaping of output)禁止URL中的字符自动转义成%[0-9]+的形式  
> 'passthrough|PT' (pass through to next handler)将重写结果运用于mod_alias  
> 'skip|S=num' (skip next rule(s))跳过下面几个规则  
> 'env|E=VAR:VAL' (set environment variable)添加环境变量  
>
因此，使用[OR]可以连接多个RewriteCond或者RewriteRule，使用[L]可以实现前述伪代码里的return效果。
# 写在后面
由于较少接触服务器配置，受限于技术积累，靠搜索引擎和有限的时间精力只能做到这一步了，但用以举一反三实现一些简单的功能应该是可以做到的。  
这样弄懂了应该可以避免以后的复制粘贴，但前面提到的玄学内容暂时无力深究了，略有遗憾。
# 参考文献  
1. [Apache HTTP Server Tutorial: .htaccess files](http://httpd.apache.org/docs/current/howto/htaccess.html)  
2. [$_SERVER](https://baike.baidu.com/item/$_SERVER/4897514)
3. [Apache .htaccess规则RewriteCond 和RewriteRule-实操解释说明](https://blog.csdn.net/cplvfx/article/details/94726074)
4. [Apache .htaccess规则说明](https://blog.csdn.net/cplvfx/article/details/94725685)