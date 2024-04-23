# SQL注入的关键点

## 注入的是什么：

sql注入的就是SQL语句，核⼼思维：想法设法去执⾏⼀条完全的sql语句，把数据带出来或把命令传进去。

程序员想要用户输入的是数据流，但如果一味的相信用户输入的数据流，当用户输入的数据流污染了程序的控制流，漏洞便产生了

## 关注点 

1.关注数据库类型，不同数据库类型所支持的函数调用不一样

2.编程语言不重要 语言的目的都是送入payload

3.产生注入不同的输入点 决定了我们的攻击向量Vector以及是否需要绕过

比如：I:能否堆叠注入  

微软sql server 和orcel比较多 MySQL较少,一条SQL语句最终会被解释成一条控制序列,而堆叠注入可以跳脱原生的控制序列

![image-20231107145546309](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107145546309.png)

II:爆数据     报错与联合查询速度>带外>布尔>时间盲注（考虑网速）



# ORM框架

ORM将数据库和对象进行映射, 由对象去操作数据库而不是SQL语句,提高了数据库的安全性但调⽤orm的错误写法也会造成漏洞

web框架或者cms⾃⾏去实现⾃⼰的orm（利⽤点， ⼀套闭源cms的orm可能实现得并不规范，产⽣漏洞）

java/MyBatis有两种变量绑定⽅式，分别是：#{}和${}  #是占位   $是拼接   $可能导致漏洞

sqlalchemy是flask最经常配套的orm框架，在许多django项⽬中也常常看到身影。也是⽬前python上⽤得最⽕的orm框架。

![image-20231107174443241](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107174443241.png)

# 可以值得参考的注入点

## 宽字节注入

使用了非英文的编码在进行编码转换时候绕过了过滤机制导致漏洞产生

数据在存储的时候⼀定是以字节存储的，但是数据解读的时候，都是以字符的标准去解读的

![image-20231107114026722](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107114026722.png)

![image-20231107114225623](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107114225623.png)

如在GBK编码字节序列中，一个字符可以由一个或两个字节表示，如果一个英文字符（ASCII编码）和一个特殊字符（如反斜杠）组合在一起，就可能形成一个有效的GBK字符，从而绕过安全过滤

数据以字节存储 以字符输出![image-20231107113859923](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107113859923.png)



## HQL注入

HQL，即Hibernate(orm框架)提供的面向对象的查询语言。它与SQL非常相似，但HQL是完全面向对象，针对Java对象而不是数据库表的,意味着，无论底层数据库是Oracle、MySQL还是SQL Server，HQL都能够正常工作。这是因为Hibernate会将HQL查询转换为针对特定数据库的SQL查询

用户输入作为HQL的一部分经过hibernate解释引擎渲染成SQL语句再放到数据库进行查询

HQL注⼊的漏洞产⽣与SQL注⼊并⽆差异，也是直接通过拼接字符串导致的注⼊。依然符合我 们的定侓，相对于SQL注⼊⽽⾔，HQL增加的挑战就是我们插⼊的数据，⾸先必须经过HQL渲染通过才能 进⼊SQL数据库获取数据。

⼀般的利⽤⽅式是，不考虑sql能够完全执⾏成功，⽽是利⽤sql报错注⼊+框架开启报 错，将有⽤的数据直接在错误回显中爆出来





## 预编译模式化的注入

### 宽字节

![image-20231107161710380](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107161710380.png)

![image-20231107161723597](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107161723597.png)

### 无法编译的输入点



#### like与in关键字

like关键字默认是不会预编译的(如果使⽤Mybatis则是预编译报错)。数据库⽅给出的原因   like预编译会造成慢查询和DOS

![image-20231107161907965](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107161907965.png)

#### 不能加引号的关键字

表名、列名、limit⼦句、order by[desc/asc]

很多开发会遗漏这个点，导致存在注⼊。 或者⼀些java的开发，Mybatis编译报错，然后他们⾃⼰去添加过滤（过滤没写好）或者不过 滤（使⽤原⽣语句），导致GG



权限的关系

**认清数据平面**

liunx root    Windows  system

1.在cms/web层⾯：web后台管理员权限>web前台会员权限>游客权限 

2.在数据库层⾯:root⽤户/sa⽤户/postgres⽤户>普通杂七杂⼋⽤户

3.在操作系统层⾯:root > 所有其他⽤户

同进程之间权限是相同的，上下级进程之间的权限是继承的，权限取决于我们的目标

![image-20231107164530046](C:\Users\西山\AppData\Roaming\Typora\typora-user-images\image-20231107164530046.png)
