# Spring中IOC和AOP

## IOC 控制反转

IOC就是将用例创建交由第三方容器来处理，



## AOP 面向切面编程

利用注解的方式让公共部分提取出来，如日志记录，权限认证。 Spring AOP或者AspectJ



# @Autowired和@Resourece

@Autowired属于Spring内置的注解，默认的注入方式为byType，会优先根据接口类型来匹配注入Bean

如果接口存在多个实现类的话，byType无法正确注入对象。