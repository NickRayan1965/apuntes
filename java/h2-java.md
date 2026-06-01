# H2 - Database Java

### properties.yaml
```yaml
  spring:
    datasource:
      url: jdbc:h2:mem:testdb
      username: sa
      password:
      driver-class-name: org.h2.Driver

    h2:
      console:
        enabled: true
        path: /h2-console

    jpa:
      hibernate:
        ddl-auto: update
      show-sql: true
```
### Detalles
- `spring.datasource.url`: 
```properties
# En memoria
jdbc:h2:mem:testdb

# En archivo
jdbc:h2:file:./data/veterinaria

# Archivo en ruta absoluta
jdbc:h2:file:C:/db/veterinaria
```
- `spring.datasource.class-name`: Spring boot puede detectarlo en automatico, se puede omitir

- `spring.h2.console.enabled`: Habilita la consola web de H2 para acceder a la base de datos desde el navegador. 
- `spring.h2.console.path`: Define la ruta de acceso a la consola web de H2. Por defecto es `/h2-console`, pero puedes cambiarlo.

### Notas
#### H2 Console
Para ingresar a la consola web de H2, abre tu navegador y ve a `http://localhost:8080/h2-console`.
y asegurate de poner en el JDBC URL: `jdbc:h2:mem:testdb` (o el que hayas configurado) y el username y password que hayas definido