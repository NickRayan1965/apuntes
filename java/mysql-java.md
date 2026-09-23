# MYSQL - Database Java

### properties.yaml
```yaml
spring:
  datasource: 
    url: jdbc:mysql://localhost:3306/operational_alerts
    username: root
    password: '1234'
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa: 
    hibernate:
      ddl-auto: none
    show-sql: true
    properties: 
      hibernate: 
        dialect: org.hibernate.dialect.MySQLDialect
```

### Dependency
```yaml
        <!-- Connector MySQL -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>   
```
#### Otra depedendencia: Spring Data JPA
```yaml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
```


