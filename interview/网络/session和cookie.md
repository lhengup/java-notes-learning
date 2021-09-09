### 共同之处：

cookie和session都是用来跟踪浏览器用户身份的会话方式。

### 工作原理：

1. Cookie的工作原理  
   （1）浏览器端第一次发送请求到服务器端  
   （2）服务器端创建Cookie，该Cookie中包含用户的信息，然后将该Cookie发送到浏览器端  
   （3）浏览器端再次访问服务器端时会携带服务器端创建的Cookie  
   （4）服务器端通过Cookie中携带的数据区分不同的用户  
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917204655188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW4xMzMzMzMzNjY3Nw==,size_16,color_FFFFFF,t_70)

2. Session的工作原理  
   （1）浏览器端第一次发送请求到服务器端，服务器端创建一个Session，同时会创建一个特殊的Cookie（name为JSESSIONID的固定值，value为session对象的ID），然后将该Cookie发送至浏览器端  
   （2）浏览器端发送第N（N>1）次请求到服务器端,浏览器端访问服务器端时就会携带该name为JSESSIONID的Cookie对象  
   （3）服务器端根据name为JSESSIONID的Cookie的value(sessionId),去查询Session对象，从而区分不同用户。
   name为JSESSIONID的Cookie不存在（关闭或更换浏览器），返回1中重新去创建Session与特殊的Cookie
   name为JSESSIONID的Cookie存在，根据value中的SessionId去寻找session对象
   value为SessionId不存在**（Session对象默认存活30分钟）**，返回1中重新去创建Session与特殊的Cookie
   value为SessionId存在，返回session对象  
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019091720521815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW4xMzMzMzMzNjY3Nw==,size_16,color_FFFFFF,t_70)

### 区别：

cookie数据保存在客户端，session数据保存在服务端。  

session  
简单的说，当你登陆一个网站的时候，如果web服务器端使用的是session，那么所有的数据都保存在服务器上，客户端每次请求服务器的时候会发送当前会话sessionid，服务器根据当前sessionid判断相应的用户数据标志，以确定用户是否登陆或具有某种权限。由于数据是存储在服务器上面，所以你不能伪造。

cookie  
sessionid是服务器和客户端连接时候随机分配的，如果浏览器使用的是cookie，那么所有数据都保存在浏览器端，比如你登陆以后，服务器设置了cookie用户名，那么当你再次请求服务器的时候，浏览器会将用户名一块发送给服务器，这些变量有一定的特殊标记。服务器会解释为cookie变量，所以只要不关闭浏览器，那么cookie变量一直是有效的，所以能够保证长时间不掉线。  

如果你能够截获某个用户的cookie变量，然后伪造一个数据包发送过去，那么服务器还是 认为你是合法的。所以，使用cookie被攻击的可能性比较大。  

如果cookie设置了有效值，那么cookie会保存到客户端的硬盘上，下次在访问网站的时候，浏览器先检查有没有cookie，如果有的话，读取cookie，然后发送给服务器。  

所以你在机器上面保存了某个论坛cookie，有效期是一年，如果有人入侵你的机器，将你的cookie拷走，放在他机器下面，那么他登陆该网站的时候就是用你的身份登陆的。当然，伪造的时候需要注意，直接copy cookie文件到 cookie目录，浏览器是不认的，他有一个index.dat文件，存储了 cookie文件的建立时间，以及是否有修改，所以你必须先要有该网站的 cookie文件，并且要从保证时间上骗过浏览器  

两个都可以用来存私密的东西，session过期与否，取决于服务器的设定。cookie过期与否，可以在cookie生成的时候设置进去。  



###  区别对比：  
(1)cookie数据存放在客户的浏览器上，session数据放在服务器上  
(2)cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗,如果主要考虑到安全应当使用session  
(3)session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，如果主要考虑到减轻服务器性能方面，应当使用COOKIE  
(4)单个cookie在客户端的限制是3K，就是说一个站点在客户端存放的COOKIE不能3K。  
(5)所以：将登陆信息等重要信息存放为SESSION;其他信息如果需要保留，可以放在COOKIE中  