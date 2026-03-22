# FarmaExpres_Micro_Login

## Auth Service

### Descripción
El servicio de autenticación (`auth-service`) es un microservicio basado en Spring Boot que maneja la autenticación y autorización de usuarios mediante JWT (JSON Web Tokens). Proporciona funcionalidades para login, registro de usuarios, cambio de contraseñas y registro de bitácora de actividades.

### Tecnologías Utilizadas
- **Java**: 17
- **Spring Boot**: 4.0.3
- **Spring Security**: Para autenticación y autorización
- **JWT**: jjwt-api (versión 0.11.5) para tokens de autenticación
- **Base de Datos**: PostgreSQL con JPA/Hibernate
- **Build Tool**: Maven
- **Otras**: Lombok (opcional), Actuator para monitoreo

### Arquitectura
El servicio sigue una arquitectura en capas:
- **Controllers**: Exposición de APIs REST
- **Services**: Lógica de negocio
- **Repositories**: Acceso a datos
- **Entities**: Modelos de datos
- **DTOs**: Objetos de transferencia de datos
- **Config**: Configuración de seguridad

### Endpoints Principales

#### Autenticación
- `POST /api/auth/login`
  - Public endpoint for user login
  - Request body:
    ```json
    {
      "email": "string",
      "password": "string"
    }
    ```
  - Response:
    ```json
    {
      "token": "jwt",
      "type": "Bearer",
      "email": "string",
      "role": "string"
    }
    ```

#### Usuarios (Requiere ROLE_ADMIN)
- `POST /api/users`
  - Creates a new user
  - Request body:
    ```json
    {
      "name": "string",
      "email": "string",
      "password": "string",
      "role": "string"
    }
    ```

    - `PUT /api/users/{id}/password`
  - Updates a user's password
  - Request body:
    ```json
    {
      "currentpassword": "string",
      "newPassword": "string"
    }
    ```

#### Bitácora (Requiere ROLE_ADMIN)
- `GET /api/binnacle`
  - Returns all binnacle records


#### Estado del Servicio
- `GET /status` - Verificar estado del servicio (público)
- `GET /actuator/health` - Health check (público)
- `GET /actuator/info` - Información del servicio (público)

### Configuración


**Variables de entorno requeridas:**
- `DB_URL`: URL de conexión a PostgreSQL (ej: `jdbc:postgresql://localhost:5432/authdb`)
- `DB_USERNAME`: Usuario de la base de datos
- `DB_PASSWORD`: Contraseña de la base de datos
- `JWT_SECRET`: Clave secreta para firmar JWT (mínimo 256 bits)

### Estructura del Proyecto
```text
auth-service/
├── src/main/java/co/edu/corhuila/auth_service/
│   ├── AuthServiceApplication.java
│   ├── Config/
│   │   └── SecurityConfig.java
│   ├── Controllers/
│   │   ├── AuthController.java
│   │   ├── BinnacleController.java
│   │   ├── StatusController.java
│   │   └── UsurControllers.java
│   ├── Service/
│   │   ├── AuthService.java
│   │   ├── JwtFilter.java
│   │   ├── JwtService.java
│   │   └── userService.java
│   ├── Entity/
│   │   ├── Binnacle.java
│   │   ├── Role.java
│   │   ├── User.java
│   │   └── UserStatus.java
│   ├── Repository/
│   │   ├── BinnacleRepository.java
│   │   ├── RoleRepository.java
│   │   └── UserRepository.java
│   ├── DTO/
│   │   ├── ApiErrorResponse.java
│   │   ├── LoginRequest.java
│   │   ├── LoginResponseDto.java
│   │   ├── UsurRequest.java
│   │   ├── UsurResponse.java
│   │   └── changePasswordRequest.java
│   └── exception/
│       └── GlobalExceptionHandler.java
├── src/main/resources/
│   └── application.yaml
├── Dockerfile
├── pom.xml
└── mvnw*

### Modelo de Datos
- **User**: id, name, email, password (bcrypt), state, role
- **Role**: idRole, name, description
- **Binnacle**: id, usurId, action, dateTime
- **UserStatus**: Asset, Blocked

### Seguridad
- **Autenticación**: JWT con expiración de 1 hora
- **Autorización**: Basada en roles (ADMIN, USER)
- **Encriptación**: BCrypt para contraseñas
- **Filtros**: JwtFilter intercepta requests para validar tokens
- **Endpoints públicos**: login, status, actuator health/info
- **Endpoints protegidos**: requieren autenticación, algunos ROLE_ADMIN

### Ejecución
1. **Requisitos previos**:
   - Java 17
   - PostgreSQL
   - Maven

2. **Configurar variables de entorno**:
   ```bash
   export DB_URL=jdbc:postgresql://localhost:5432/authdb
   export DB_USERNAME=your_username
   export DB_PASSWORD=your_password
   export JWT_SECRET=your_super_secret_key_min_256_bits
   ```

3. **Ejecutar**:
   ```bash
   cd auth-service
   ./mvnw spring-boot:run
   ```

4. **Con Docker**:
   ```bash
   docker build -t auth-service .
   docker run -p 8081:8081 --env-file .env auth-service
   ```

### Pruebas
El proyecto incluye tests unitarios. Ejecutar con:
```bash
./mvnw test
```
