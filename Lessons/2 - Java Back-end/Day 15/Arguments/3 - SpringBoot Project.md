# SpringBoot Project

## Learning Objectives

- Generate the exact Spring Boot resource-server project used in the lesson.
- Configure stateless JWT validation before any request reaches the controller.
- Compare one public endpoint with one protected endpoint in the test controller.

## Create the project from the Devbox shell

Create a fresh application so the only mechanism under test is authentication. Do not copy code from another project.

From the folder containing `devbox.json`, enter the Devbox shell and run:

```sh file:"Create JavaOidc"
spring init \
  --type=gradle-project \
  --language=java \
  --boot-version=4.1.1 \
  --group-id=dev.wows.buk \
  --artifact-id=JavaOidc \
  --name=JavaOidc \
  --package-name=dev.wows.buk.JavaOidc \
  --java-version=21 \
  --dependencies=web,devtools,postgresql,lombok,oauth2-resource-server \
  --extract \
  JavaOidc
```

The command creates and extracts `JavaOidc`, including the Gradle Wrapper. `web` provides the controller and HTTP runtime. `oauth2-resource-server` provides Spring Security's bearer-token support. The generated PostgreSQL and Lombok dependencies are not used by this authentication-only example.

## Configure the application

Replace `JavaOidc/src/main/resources/application.yaml` with this complete file:

```yaml file:application.yaml
server:
  port: 4000
spring:
  web:
    error:
      include-message: always
      include-binding-errors: always
      include-stacktrace: never
      include-exception: false
  datasource:
    url: jdbc:postgresql://localhost:5432/booleandb
    username: postgres
    password: ""
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8080/realms/springboot-realm-1
```

The application listens on port `4000`. The Keycloak URLs use the `springboot-realm-1` realm created in the dashboard. `resourceserver` belongs directly under `oauth2`; do not place it under `client`, because `JavaOidc` validates bearer tokens rather than performing OAuth2 login. The datasource and JPA entries are present in the complete configuration file but are not exercised by the two test endpoints.

## Put security before the controller

Create `JavaOidc/src/main/java/dev/wows/buk/JavaOidc/security/SecurityConfig.java`:

```java file:SecurityConfig.java
package dev.wows.buk.JavaOidc.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain web(HttpSecurity http) throws Exception {

        return http
                .csrf(AbstractHttpConfigurer::disable)
                .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/public/**").permitAll()
                        .anyRequest().authenticated()
                )
                .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
                .build();
    }

    @Bean
    JwtDecoder jwtDecoder() {

        return NimbusJwtDecoder.withJwkSetUri(
            "http://localhost:8080/realms/springboot-realm-1/protocol/openid-connect/certs"
        ).build();
    }
}
```

`SecurityFilterChain` is the ordered set of security rules applied before a controller method runs.

- CSRF protection is disabled for this stateless bearer-token API.
- `STATELESS` tells Spring Security not to create or reuse an HTTP login session.
- Requests under `/public/**` are permitted without authentication.
- Every other request must be authenticated.
- The resource server reads a JWT bearer token with Spring Security's default JWT handling.
- `JwtDecoder` obtains Keycloak's public signing keys from the realm's JWK endpoint.

## Add only the observable endpoints

Create `JavaOidc/src/main/java/dev/wows/buk/JavaOidc/controller/AuthTestController.java`:

```java file:AuthTestController.java
package dev.wows.buk.JavaOidc.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AuthTestController {

    @GetMapping("public/test")
    public String getPublicHello() {

        return "Public Hello, World!";
    }

    @GetMapping("test")
    public String getPrivateHello() {

        return "Private Hello, World";
    }
}
```

There is one controller and two endpoints:

| Request | Expected access | Response body after access is granted |
| --- | --- | --- |
| `GET /public/test` | No token required | `Public Hello, World!` |
| `GET /test` | Valid bearer token required | `Private Hello, World` |

Add no other controllers, domain layers or endpoints. This project stops at `SecurityConfig` and `AuthTestController` so authentication remains the only behaviour under test.

## Run the resource server

Keep the Keycloak Devbox service running. In another Devbox shell:

```sh file:"Run JavaOidc"
cd JavaOidc
./gradlew bootRun
```

First request `GET http://localhost:4000/public/test` without authorisation. It should return `200 OK`. Then request `GET http://localhost:4000/test` without authorisation. Spring Security should return `401 Unauthorized` before `getPrivateHello()` runs.

The next document obtains a token in Postman and proves that the same protected request succeeds when the caller is authenticated.

---

# Links
![[Lessons/2 - Java Back-end/Day 15/__blocks/Links]]
