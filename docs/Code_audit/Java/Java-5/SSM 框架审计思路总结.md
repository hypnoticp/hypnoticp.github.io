SSM 框架的审计思路其实就是代码的执行思路。

与审计非 SSM 框架代码的主要区别在于 SSM 框架的各种 XML 配置，注解配置需要用户根据 XML 中的配置和注解来查看代码的执行路径、SSM 框架中常见的注解和注解中的属性，以及常见的标签和标签的各个属性。

审计漏洞的方式与正常的 Java 代码审计没有区别，网上有很多非常优秀的 Java代码审计文章，关于每个漏洞的审计方式写得都非常全面，我们需要做的只是将其移植到 SSM 框架的审计中来。明白 SSM 的执行流程后自然就明白怎样在 SSM 框架中跟踪参数，例如刚刚介绍的 XSS 漏洞。我们根据 XML 中的配置和注解中的配置找到了 MyBatis 的 mapper.xml 这个映射文件，以及最终执行的以下命令。

```
insert into ssmbuild.books(bookName,bookCounts,detail) 
values (#{bookName}, #{bookCounts}, #{detail})
```

观察这个 SQL 语句，发现传入的 books 参数直到 SQL 语句执行的前一刻都没有经过任何过滤处理，所以此处插入数据库的参数自然是不可信的脏数据。再次查询这条数据并返回到前端时就非常可能造成存储型 XSS 攻击。

在审计这类漏洞时，最简单的方法是先在 web.xml 中查看有没有配置相关的过滤器，如果有则查看过滤器的规则是否严格，如果没有则很有可能存在漏洞。

最后补充一下 MyBaits 中的预编译知识。在非预编译的情况下，用户每次执行SQL 都需要将 SQL 和参数拼接在一起，然后传递给数据库编译执行，这种采用拼接的方式非常容易产生 SQL 注入漏洞，用户可以使用 filter 对参数进行过滤来避免产生SQL 注入。

而在预编译的情况下，程序会提前将 SQL 语句编译好，程序执行时只需要将传递进来的参数交由数据库进行操作即可。此时不论传递进来的参数是什么，都不会被当作 SQL 语句的一部分，因为真正的 SQL 语句已经提前被编译好了，所以即使不过滤也不会产生 SQL 注入这类漏洞，以下面 mapper.xml 中的 SQL 语句为例。

```
insert into ssmbuild.books(bookName,bookCounts,detail) 
values (#{bookName}, #{bookCounts}, #{detail})
```

#{bookName}这种形式就是采用了预编译的形式传参。

```
insert into ssmbuild.books(bookName,bookCounts,detail) 
values ('${bookName}','${bookCounts}', '${detail}')
```

```
而'${bookName}'这种写法没有使用预编译的形式传递参数，此时如果不对传入
的参数进行过滤和校验，就会产生 SQL 注入漏洞，'${xxxx}'和#{xxxx}其实就是 JDBC
的 Statement 和 PreparedStatement 对象。
```