# Flyway
* Para control de versiones de bases de datos y migraciones.
* Tabla metadatos flyway_schema_history
* Checksum
* <b>Baseline</b>: registra el estado actual de la db como la versión inicial de la migración (V1)
* repair
 * src/main/resources/db/migration/
#### Convencion de nombres
  - <b>Versionadas</b>: "V{Version}__{DescripcionCualquiera}.sql"
    
  - <b>Repetibles</b>: R__{DescripcionCualquiera}.sql
#### Configurar proyecto
  Agregar el core y la extension
  ```xml
  <dependency>
      <groupId>org.flywaydb</groupId>
      <artifactId>flyway-core</artifactId>
  </dependency>
  <dependency>
      <groupId>org.flywaydb</groupId>
      <artifactId>flyway-mysql</artifactId>
  </dependency>
  ```

  Configurar application.yml
  ```yaml
  spring:
  datasource:
    url: jdbc:mysql://localhost:3306/crm_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
    ### Este usuario puede ser solo DML
    username: root
    password: rootpassword
    driver-class-name: com.mysql.cj.jdbc.Driver

  flyway:
    enabled: true # Opcional, por defecto true
    locations: classpath:db/migration # Opcional
    baseline-on-migrate: true # Para conectarse a db ya existente y registrar la version inicial de migracion
    baseline-version: 1

    ### Opcionalemente y recomendablemente podemos configurar un propio usuario para flyway y con permisos DDL y DML sobre la base de datos.
    url: jdbc:mysql://localhost:3306/crm_db
    user: flyway_user
    password: flyway_password
  ```
#### Baseline
  Cuando nuestra base da datos no esta desde 0 tenemos que activar las siguientes propiedades
  ```yaml
  flyway:
    baseline-on-migrate: true
    baseline-version: 1
  ```
  baseline-version: 1 indica que solo ejecute las migraciones que sean mayores a 1, es decir que no ejecute la V1__init.sql porque ya tenemos la base de datos creada y no queremos que se ejecute ese script.

  Esta configuracion no afecta a cuando estamos en desarrollo y queremos crear la base de datos desde 0, ya que si no existe la base de datos flyway la creara y ejecutara todas las migraciones.
#### Comandos y Solución de Fallos
  - **Fallo por Checksum alterado en Script V ya aplicado:**
  No modifiques scripts `V` antiguos en producción.
  Si ocurre un error y se registra en flyway_schema_history un success = 0 para una migracion (archivo V), la app no arrancará, hay que seguir 3 pasos para resolverlo:
  ##### 1 . Revisar el estado de la db manualmente y revertir cambios si es necesario.
  ##### 2 . Ejecutar flyway:repair
  Esto elimina el registro de la migración fallida y permite que se vuelva a ejecutar.
  ```bash
  # Opción A: Vía CLI de Maven (pasando credenciales DDL si las tienes separadas)
  ./mvnw flyway:repair -Dflyway.url=jdbc:mysql://localhost:3306/crm_db -Dflyway.user=root -Dflyway.password=rootpassword

  # Opción B: Si tienes el plugin de Flyway configurado en el pom.xml
  ./mvnw flyway:repair
  ```
  ##### 3 . Corregir el script sql fallido

  Ahora se podra ejecutar

#### Referencias:
    https://medium.com/@AlexanderObregon/using-spring-boot-with-flyway-to-manage-database-migrations-8180ce0c9230

