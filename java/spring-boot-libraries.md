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
  
### Documentación OpenAPI / Swagger
  ```xml
  <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
      <version>2.5.0</version>
  </dependency>
  ```
  ##### Configuración general
  OpenApiConfig.java en infrastructure/config



## 2. Spring WebFlux