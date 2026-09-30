# Tecnicas Spring boot

## 1. Inserts Masivos (Batch Inserts)
### 1.1 Usando NamedParameterJdbcTemplate con .batchUpdate
  #### Requisitos
  ##### a.   
  Tenemos que tener instalado spring-boot-starter-jdbc o spring-boot-starter-data-jpa (que ya lo trae).
  ##### b.
  Configurar rewriteBatchedStatements en el application.properties o application.yml para activar el batching de inserts en MySQL. 
  Esto tranforma múltiples inserts en un solo insert con múltiples listas de valores.
  Si es un update o delete los manda en un solo round-trip a la BD. 
  ```properties
  spring.datasource.url=jdbc:mysql://localhost:3306/tu_base_de_datos?rewriteBatchedStatements=true
  ```
  #### Ejemplo

  ```java
  package com.adcapricornio.operational_alerts.infrastructure.adapters.out.persistence;

  import java.util.List;
  import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
  import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
  import org.springframework.jdbc.core.namedparam.SqlParameterSource;
  import org.springframework.stereotype.Repository;
  import org.springframework.transaction.annotation.Transactional;

  @Repository
  public class BatchAlertRepository {

      private final NamedParameterJdbcTemplate namedParameterJdbcTemplate;

      public BatchAlertRepository(NamedParameterJdbcTemplate namedParameterJdbcTemplate) {
          this.namedParameterJdbcTemplate = namedParameterJdbcTemplate;
      }

      private static final String BATCH_INSERT_SQL = """
          INSERT INTO tbl_alerta_configuracion (
              CODIGO_USUARIO, 
              CODIGO_TIPO_ALERTA, 
              SONIDO_NOTIFICACION, 
              SONIDO_INTERMITENTE
          ) VALUES (
              :userId, 
              :alertTypeId, 
              :notificationSound, 
              :intermittentSound
          )
      """;

      @Transactional
      public void saveAllBatch(List<AlertTypeConfigCommand> items) {
          if (items.isEmpty()) return;

          // 1. Mapear la lista de objetos a un array de parámetros
          SqlParameterSource[] batchArgs = items.stream()
              .map(item -> new MapSqlParameterSource()
                  .addValue("userId", item.getUserId())
                  .addValue("alertTypeId", item.getAlertTypeId())
                  .addValue("notificationSound", item.getNotificationSound())
                  .addValue("intermittentSound", item.getIntermittentSound())
              )
              .toArray(SqlParameterSource[]::new);

          // 2. Ejecutar un solo round-trip a BD
          namedParameterJdbcTemplate.batchUpdate(BATCH_INSERT_SQL, batchArgs);
      }
  }
  ```
### 1.2 Usando Sql Navito o en Store Procedures
  ```sql
  INSERT INTO tbl_alerta_configuracion (CODIGO_USUARIO, CODIGO_TIPO_ALERTA)
  SELECT userId, alertTypeId
  FROM JSON_TABLE(pTxt_datos, '$[*]' COLUMNS (
      userId INT PATH '$.userId',
      alertTypeId INT PATH '$.alertTypeId'
  )) AS jt;
  ```