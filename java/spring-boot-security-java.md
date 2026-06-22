# Spring boot Security

## 1 Dependencias
```yml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
  <scope>compile</scope>
</dependency>

<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-api</artifactId>
  <version>0.12.5</version>
</dependency>

<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-impl</artifactId>
  <version>0.12.5</version>
  <scope>runtime</scope>
</dependency>

<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-jackson</artifactId>
  <version>0.12.5</version>
  <scope>runtime</scope>
</dependency>
```

## 2 Configuración de seguridad
  ### 2.1 Security Config
  Clase de configuración de seguridad de Spring, donde se configuran las reglas de seguridad, los filtros y los manejadores de errores.
  - desactivamos csrf que es para las sessiones
  - configuramos como stateless
  - configuramos los endpoints publicos y privados
  - configuramos authentication provider para que use nuestro CustomUserDetailsService y el password encoder
  - configuramos el filtro de jwt para q se ejecute antes de la autenticacion spring
  - configuramos los manejadores de errores para que devuelvan nuestros propios mensajes de error en formato json para los 401 y 403
  - el autentication manager se configura para poder inyectarlo y usarlo en el login
  ```java
  package com.nickdev.vet.config;

  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.security.authentication.AuthenticationManager;
  import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
  import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
  import org.springframework.security.config.annotation.web.builders.HttpSecurity;
  import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
  import org.springframework.security.config.http.SessionCreationPolicy;

  import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
  import org.springframework.security.crypto.password.PasswordEncoder;

  import org.springframework.security.web.SecurityFilterChain;
  import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

  import com.nickdev.vet.model.Role;
  import com.nickdev.vet.security.CustomAccessDeniedHandler;
  import com.nickdev.vet.security.JwtAuthenticationEntryPoint;
  import com.nickdev.vet.security.JwtRequestFilter;
  import com.nickdev.vet.service.CustomUserDetailsService;

  import lombok.RequiredArgsConstructor;

  @Configuration
  @EnableWebSecurity
  @RequiredArgsConstructor
  public class SecurityConfig {

    private final JwtRequestFilter jwtRequestFilter;
    private final CustomUserDetailsService customUserDetailsService;
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    private final CustomAccessDeniedHandler customAccessDeniedHandler;
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) {
      http
      .csrf(csrf -> csrf.disable())
      .sessionManagement(session ->
                          session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
      .exceptionHandling(ex -> 
        ex.authenticationEntryPoint(jwtAuthenticationEntryPoint)
        .accessDeniedHandler(customAccessDeniedHandler)
      )
      .authorizeHttpRequests(auth -> 
        auth.requestMatchers("/api/v1/public/**").permitAll()
        .requestMatchers("/api/v1/users/login").permitAll()
        .requestMatchers("/api/v1/users").hasRole(Role.ADMIN.name())
        .anyRequest().authenticated()
      )
      .authenticationProvider(authenticationProvider())
      .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class)
      ;
      return http.build();
    }
    @Bean
    DaoAuthenticationProvider authenticationProvider() {

        DaoAuthenticationProvider provider =
                new DaoAuthenticationProvider(customUserDetailsService);

        provider.setPasswordEncoder(passwordEncoder());

        return provider;
    }
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @Bean
    AuthenticationManager authenticationManager(
            AuthenticationConfiguration config)
            throws Exception {

        return config.getAuthenticationManager();
    }

  }
  ```
  ### 2.2 Jwt Util 
  Clase para configurar el manejo de tokens JWT, incluyendo generación y validación de tokens.
  ```java
package com.nickdev.vet.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;

@Component
public class JwtUtil {

    private final SecretKey key;
    private final long expirationTime;

    public JwtUtil(
            @Value("${jwt.secret}") String secret,
            @Value("${jwt.expiration}") long expirationTime
    ) {
        this.key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
        this.expirationTime = expirationTime;
    }

    public String generateJwt(String username) {
        Date now = new Date();
        return Jwts.builder()
                .subject(username)
                .issuedAt(now)
                .expiration(new Date(now.getTime() + expirationTime))
                .signWith(key)
                .compact();
    }

    public Claims validateAndGetClaims(String token) {
        return Jwts.parser()
                .verifyWith(key)
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }
}
  ```
  ### 2.3 CustomUserDetails
  Clase que implementa `UserDetails` para representar el usuario pero agregandole una capa de nuestra propia entidad de usuario
  ```java
  package com.nickdev.vet.security;

  import java.util.Collection;

  import org.jspecify.annotations.Nullable;
  import org.springframework.security.core.GrantedAuthority;
  import org.springframework.security.core.userdetails.UserDetails;

  import com.nickdev.vet.model.SystemUser;

  import lombok.Builder;
  import lombok.Getter;
  import lombok.Setter;

  @Getter
  @Setter
  @Builder
  public class CustomUserDetails implements UserDetails{
    private SystemUser user;
    private Collection<? extends GrantedAuthority> authorities;
    @Override
    public @Nullable String getPassword() {
      return user.getPassword();
    }
    @Override
    public String getUsername() {
      return user.getUsername();
    }
  }
  ```
  ### 2.4 CustomUserDetailsService
  Clase que implementa `UserDetailsService` para configurarla en Spring Security, permitiendole hacer la autenticación de usuarios por username a través de nuestra base de datos y con nuestra logica de datos permitiendo usar nuestro CustomUserDetails.
  ```java
  package com.nickdev.vet.service;

  import java.util.List;

  import org.springframework.security.core.authority.SimpleGrantedAuthority;
  import org.springframework.security.core.userdetails.UserDetails;
  import org.springframework.security.core.userdetails.UserDetailsService;
  import org.springframework.security.core.userdetails.UsernameNotFoundException;
  import org.springframework.stereotype.Service;

  import com.nickdev.vet.model.SystemUser;
  import com.nickdev.vet.repository.UserRepository;
  import com.nickdev.vet.security.CustomUserDetails;

  import lombok.RequiredArgsConstructor;

  @Service
  @RequiredArgsConstructor
  public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
      SystemUser user = userRepository.findByUsername(username)
        .orElseThrow(() -> new UsernameNotFoundException("User not found with username: " + username));
      
      return CustomUserDetails.builder()
        .user(user)
        .authorities(
          List.of(
            new SimpleGrantedAuthority("ROLE_" + user.getRole().name())
          )
        )
        .build();
    }
    
  }
  ```
  ### 2.5 Jwt Request Filter
  Manejador que se ejecuta en cada request para validar el jwt en caso venga y setear el objeto de autenticacion en el contexto de seguridad de Spring.
  ```java
  package com.nickdev.vet.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import io.jsonwebtoken.Claims;

import java.io.IOException;

@Component
@RequiredArgsConstructor
public class JwtRequestFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {
    
        final String authHeader = request.getHeader("Authorization");
    
    
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        if (SecurityContextHolder.getContext().getAuthentication() != null) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);

        Claims claims = jwtUtil.validateAndGetClaims(token);

        String username = claims.getSubject();

        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        UsernamePasswordAuthenticationToken authentication =
            new UsernamePasswordAuthenticationToken(
                userDetails,
                null,
                userDetails.getAuthorities()
            );

        SecurityContextHolder.getContext()
            .setAuthentication(authentication);    
        
    
        filterChain.doFilter(request, response);
    }
}
  ```
  ### 2.6 Authentication EntryPoint
  Es un manejador que se ejecuta cuando el usuario no esta autenticado y deberia devolver un error 401.
  ```java
  package com.nickdev.vet.security;

  import java.io.IOException;

  import org.springframework.http.HttpStatus;
  import org.springframework.http.MediaType;
  import org.springframework.security.core.AuthenticationException;
  import org.springframework.security.web.AuthenticationEntryPoint;
  import org.springframework.stereotype.Component;

  import com.nickdev.vet.exception.ApiError;

  import jakarta.servlet.http.HttpServletRequest;
  import jakarta.servlet.http.HttpServletResponse;
  import lombok.RequiredArgsConstructor;
  import tools.jackson.databind.ObjectMapper;

  @Component
  @RequiredArgsConstructor
  public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {

      private final ObjectMapper objectMapper;

      @Override
      public void commence(HttpServletRequest request, HttpServletResponse response,
                            AuthenticationException authException) throws IOException {

          ApiError error = ApiError.unauthorized("Invalid or missing JWT token");

          response.setStatus(HttpStatus.UNAUTHORIZED.value());
          response.setContentType(MediaType.APPLICATION_JSON_VALUE);
          response.getWriter().write(objectMapper.writeValueAsString(error));
      }
  }
  ```
  ### 2.7 Access Denied Handler
  Es un manejador que se ejecuta cuando el usuario esta autenticado pero no tiene permisos para acceder a un recurso, deberia devolver un error 403.
  ```java
  package com.nickdev.vet.security;

  import java.io.IOException;

  import org.springframework.http.HttpStatus;
  import org.springframework.http.MediaType;
  import org.springframework.security.access.AccessDeniedException;
  import org.springframework.security.web.access.AccessDeniedHandler;
  import org.springframework.stereotype.Component;

  import com.nickdev.vet.exception.ApiError;

  import jakarta.servlet.http.HttpServletRequest;
  import jakarta.servlet.http.HttpServletResponse;
  import lombok.RequiredArgsConstructor;
  import tools.jackson.databind.ObjectMapper;

  @Component
  @RequiredArgsConstructor
  public class CustomAccessDeniedHandler implements AccessDeniedHandler {

      private final ObjectMapper objectMapper;

      @Override
      public void handle(HttpServletRequest request, HttpServletResponse response,
                          AccessDeniedException accessDeniedException) throws IOException {

        ApiError error = ApiError.forbidden("You do not have permission to access this resource");

        response.setStatus(HttpStatus.FORBIDDEN.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.getWriter().write(objectMapper.writeValueAsString(error));
      }
  }
  ```

#### Notas
#### ApiError class
```java
package com.nickdev.vet.exception;

import lombok.Builder;
import lombok.Value;

import java.time.Instant;

import org.springframework.http.HttpStatus;

@Value
@Builder
public class ApiError{
  @Builder.Default
  Instant timestamp = Instant.now();

  @Builder.Default
  int status = HttpStatus.INTERNAL_SERVER_ERROR.value();

  @Builder.Default
  String error = "Internal Server Error";

  @Builder.Default
  String message = "An unexpected error occurred";

  public static ApiError of(HttpStatus status, String message) {
    return ApiError.builder()
            .status(status.value())
            .error(status.getReasonPhrase())
            .message(message)
            .build();
  }
  public static ApiError unauthorized(String message) {
    return of(HttpStatus.UNAUTHORIZED, message);
  }
  public static ApiError forbidden(String message) {
    return of(HttpStatus.FORBIDDEN, message);
  }
  public static ApiError notFound(String message) {
    return of(HttpStatus.NOT_FOUND, message);
  }
  public static ApiError badRequest(String message) {
    return of(HttpStatus.BAD_REQUEST, message);
  }
  public static ApiError internalServerError(String message) {
    return of(HttpStatus.INTERNAL_SERVER_ERROR, message);
  }

}
```
  
