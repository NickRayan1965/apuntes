## Puntos a Trabajar o Estudiar
1. ✅ Validación de payloads (`spring-boot-starter-validation` y `@Valid`). 
2. ✅ Documentación OpenAPI / Swagger (`springdoc-openapi`).
3. Métricas y cobertura para Sonar (`jacoco-maven-plugin`).
4. ✅ Operaciones insert por lote (`batchUpdate` con `NamedParameterJdbcTemplate` o batching JPA).
5. ✅ Centralizar configuración de CORS en `CorsConfig` (eliminar `@CrossOrigin` con wildcard `*`).
6. Configurar Timeouts (`connectTimeout`, `readTimeout`), `ErrorDecoder` y/o tolerancia a fallos con Resilience4j en clientes Feign.
7. ✅ Dominar estándares JPA/Hibernate (`JpaRepository`, JPQL, `@Procedure`) y `SimpleJdbcCall` como alternativa idiomática a JDBC puro.
8. Pruebas unitarias de controladores con `@WebMvcTest` y `MockMvc`.
9. Creación de `Dockerfile` multi-stage y definición básica de `Jenkinsfile` (pipeline CI/CD).
10. Implementar propagación del token JWT hacia otros microservicios mediante `RequestInterceptor` en Feign.
11. Implementar health checks y métricas con Spring Boot Actuator (`/actuator/health`).
12. ✅ Refactorizar `RegisterAlertUseCaseImpl` para eliminar clases web de infraestructura (`AlertResponse`, `AlertRestMapper`) de la capa de aplicación. 
13. ✅ Asegurar atomicidad transaccional agregando `@Transactional` en casos de uso con múltiples escrituras.
14. ✅ Reemplazar `@RequestParam Integer userId` por la anotación personalizada `@RequestUserId` en endpoints de eliminación y reactivación. 
15. ✅ Corregir `GlobalExceptionHandler` para capturar `MethodArgumentNotValidException` (en lugar de `WebExchangeBindException`) y sanitizar el error 500 para no exponer `ex.getMessage()`.
16. Implementar interceptor o filtro para propagar `Trace-ID` / `Correlation-ID` en logs (MDC) y cabeceras HTTP.
17. ✅ Limpiar dependencias del `pom.xml` (remover `spring-boot-starter-data-jpa` si solo se usa JDBC para SPs).
18. ✅ Versionado y migración de BD automatizada con Flyway o Liquibase.
19. Implementación de caché con Spring Cache / Redis (`@Cacheable`, `@CacheEvict`).
20. Conceptos de comunicación asíncrona / event-driven con Apache Kafka o RabbitMQ (productores/consumidores vs Feign).

## Puntos a tomar en cuenta
1. En tests unitarios de casos de uso (hexagonal) no se debe levantar el contexto de Spring (`@SpringBootTest`); se usa `@ExtendWith(MockitoExtension.class)` para pruebas rápidas y aisladas.
2. Idempotencia en APIs: diseñar endpoints `PUT`, `DELETE` y reintentos para que no dupliquen efectos si la red falla.
3. Manejo de configuraciones por entorno (12-Factor App): nunca hardcodear URLs ni credenciales; consumir via variables de entorno (`ENV`) o perfiles de Spring.
4. Estrategia de Gitflow y SemVer: nombrar ramas (`feature/`, `bugfix/`, `release/`) y realizar commits semánticos (`feat:`, `fix:`, `chore:`).