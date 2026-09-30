# Librerias Spring boot

## 1. Spring Web
### 1.1 spring-boot-starter-validation
  ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
  ```
  ##### Todos reciben el parametro message
  - @NotBlank: no nulo, longitud > 0 y no solo espacios en blanco.
  - @NotEmpty: no nulo y longitud > 0, para colecciones y arrays.
  - @Size: longitud mínima y máxima de cadenas, colecciones o arrays.
  - @Valid + @NotNull: para validar objetos anidados.
  ##### Activar en el RestController
  @Valid en el parámetro del request activa el interceptor de validación, que lanza MethodArgumentNotValidException si hay errores de validación. 
  ```java
  @PostMapping
  @ResponseStatus(HttpStatus.CREATED)
  public BaseResponse<OperationalFlowResponse> create(
          @RequestUserId Integer userId,
          @Valid @RequestBody CreateOperationalFlowRequest request) { // <-- @Valid activa el interceptor
      var command = mapper.toCommand(request);
      command.setUserId(userId);
      var created = createUseCase.exec(command);
      return BaseResponse.success(mapper.toResponse(created));
  }
  ```
  ##### GlobalExceptionHandler
  Usar esto en un @ControllerAdvice
  ```java
  @ExceptionHandler(MethodArgumentNotValidException.class)
  public ResponseEntity<BaseResponse<Void>> handleValidationExceptions(MethodArgumentNotValidException ex) {
      Map<String, String> errors = new HashMap<>();
      
      ex.getBindingResult().getFieldErrors().forEach(error -> 
          errors.put(error.getField(), error.getDefaultMessage())
      );

      return ResponseEntity.status(HttpStatus.BAD_REQUEST)
              .body(BaseResponse.error("Error de validación en los campos enviados", errors, HttpStatus.BAD_REQUEST.value()));
  }
  ```
  
### 1.2 Documentación OpenAPI / Swagger
  ```xml
  <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
      <version>2.5.0</version>
  </dependency>
  ```
  ##### Hacer publico el endpoint de documentación
  ```java
    .requestMatchers("/v3/api-docs/**").permitAll()
    .requestMatchers("/swagger-ui/**").permitAll()
  ```
  ##### Configuración general
  OpenApiConfig.java en infrastructure/config
  ```java
    package com.adcapricornio.operational_alerts.infrastructure.config;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import io.swagger.v3.oas.models.Components;
    import io.swagger.v3.oas.models.OpenAPI;
    import io.swagger.v3.oas.models.info.Contact;
    import io.swagger.v3.oas.models.info.Info;
    import io.swagger.v3.oas.models.security.SecurityRequirement;
    import io.swagger.v3.oas.models.security.SecurityScheme;
    import io.swagger.v3.oas.models.security.SecurityScheme.Type;

    @Configuration
    public class OpenApiConfig {
        private static final String SECURITY_SCHEME_NAME = "BearerAuth";
        @Bean
        public OpenAPI customOpenAPI() {
            return new OpenAPI()
                .info(
                    new Info()
                        .title("Operational Alerts API")
                        .version("1.0.0")
                        .description("Microservicio de gestion de Alertas Operacionales")
                        .contact(
                            new Contact()
                                .name("Nick Rayan")
                                .email("nickcerron@gmail.com")
                        )
                )
                // 1. APlicacion global: aplica la regla atodos los endpoints
                .addSecurityItem(
                    new SecurityRequirement().addList(SECURITY_SCHEME_NAME)
                )
                .components(
                    // Esto hace que salga en el apartado Available authorizations de Swagger UI la opcion para ingresar el token
                    // y de paso se lo enviamos automaticamenta a todos los endpoints
                    new Components().addSecuritySchemes(
                        // 2. Definicion del esquema de seguridad
                        SECURITY_SCHEME_NAME, 
                        
                        new SecurityScheme()
                            .name(SECURITY_SCHEME_NAME)
                            .type(Type.HTTP)
                            .scheme("bearer")
                            .bearerFormat("JWT")
                    )
                );
        }
    }
  ```
  ##### Visualización
  Con las configuraciones Springdoc ya genera la documentación OpenAPI y Swagger UI automáticamente. Se puede acceder:
    - http://localhost:5000/swagger-ui/index.html
    - http://localhost:5000/v3/api-docs
  
  ##### Decoradores
  - @Tag(name, description): para agrupar endpoints en Swagger UI. [Class level]
  - @Operation(summary, description): para documentar un endpoint. [Method level]
  - @ApiResponses(value = { @ApiResponse(responseCode, description, content) }): para documentar respuestas de un endpoint. [Method level]
  - @Schema(description, example): para documentar un modelo de datos. [Class level and Field level]
  
### 1.3 Flyway [Mysql] (spring 3.3.5)
  Para control de versiones de bases de datos y migraciones.
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
  #### Migraciones
  Flyway por defecto busca en la ruta `src/main/resources/db/migration`.
  Podemos crear los scripts con la nomenclatura `V<version>__<description>.sql`, por ejemplo:
  - V1__init_crm_tables.sql
  #### Configuración
  
## 2. Spring WebFlux