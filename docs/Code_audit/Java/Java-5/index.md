# Java EE 开发框架安全审计

随着 Java Web 技术的不断发展，Java 开发 Web 项目由最初的单纯依靠 Servlet（在Java 代码中输出 HTML）慢慢演化出了 JSP（在 HTML 文件中书写 Java 代码）。虽然JSP 的出现在很大程度上简化了开发过程和减少了代码量，但还是对开发人员不够友好，所以慢慢地又出现了众多知名的开源框架，如 Struts2、Sping、Spring MVC、Hibernate和 MyBatis 等。目前很多成熟的大型项目在开发过程中都使用这些开源框架，而框架的本质是对底层信息的进一步封装，目的是使开发人员将更多的精力集中在业务逻辑中。针对框架的审计则需要我们对框架本身的执行流程有一定程度的了解，根据框架的执行流程逐步追踪，从而发现隐藏在项目代码中的种种安全隐患。


+ [SSM 框架审计技巧](SSM/index.md)
+ [审计的重点——filter 过滤器](Filter/index.md)
+ [SSM 框架审计思路总结](SSM 框架审计思路总结.md)
+ [SSH 框架审计技巧](SSH/index.md)
+ [Spring Boot 框架审计技巧](Spring Boot/index.md)
+ [开发框架使用不当范例(S2-001 漏洞原理分析)](s2-001/index.md)