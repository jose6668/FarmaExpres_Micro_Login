# FarmaExpres Micro Login

Microservicio de autenticacion y gestion de usuarios para el ecosistema FarmaExpres. Este proyecto esta implementado con Spring Boot y expone endpoints para login, administracion de usuarios, bitacora de eventos y verificacion de estado del servicio.

## Contenido

- API de autenticacion con JWT
- Administracion de usuarios y roles
- Cambio de contrasena, bloqueo y desbloqueo de cuentas
- Registro de eventos en bitacora
- Endpoints de estado y salud

## Stack tecnologico

- Java 17
- Spring Boot 4.0.3
- Spring Security
- Spring Data JPA
- Spring Validation
- PostgreSQL
- JWT con `jjwt` 0.11.5
- Maven Wrapper
- Spring Boot Actuator

## Estructura del proyecto

```text
FarmaExpres_Micro_Login/
|-- README.md
`-- auth-service/
    |-- Dockerfile
    |-- mvnw
    |-- mvnw.cmd
    |-- pom.xml
    `-- src/
        |-- main/
        |   |-- java/co/edu/corhuila/auth_service/
        |   |   |-- AuthServiceApplication.java
        |   |   |-- Config/
        |   |   |-- Controllers/
        |   |   |-- DTO/
        |   |   |-- Entity/
        |   |   |-- Repository/
        |   |   |-- Service/
        |   |   |-- Validation/
        |   |   `-- exception/
        |   `-- resources/
        |       `-- application.yaml
        `-- test/
            |-- java/
            `-- resources/
```

## Arquitectura

El microservicio sigue una arquitectura por capas:

- `Controllers`: exponen la API REST.
- `Service`: centraliza la logica de negocio.
- `Repository`: acceso a datos con Spring Data JPA.
- `Entity`: modelo persistente del dominio.
- `DTO`: objetos de entrada y salida de la API.
- `Validation`: validadores manuales para nombre y correo.
- `exception`: manejo centralizado de errores.

## Modelo de dominio

### User

Representa un usuario autenticable del sistema.

- `id`: identificador del usuario.
- `name`: nombre del usuario.
- `email`: correo unico.
- `password`: contrasena cifrada con BCrypt.
- `state`: estado de la cuenta.
- `role`: rol asignado.

### Role

Rol funcional asignado a un usuario.

- `idRole`
- `name`
- `description`

Roles esperados por la configuracion de seguridad:

- `ADMIN`
- `AUDITOR`
- `FARMACEUTICO`

### Binnacle

Bitacora de eventos del sistema.

- `id`
- `userId`
- `action`
- `dateTime`

Ejemplos de eventos registrados:

- `LOGIN_EXITOSO`
- `LOGIN_FALLIDO - correo`
- `CREACION_USUARIO`
- `ACTUALIZACION_USUARIO`
- `CAMBIO_PASSWORD`
- `BLOQUEO_USUARIO`
- `DESBLOQUEO_USUARIO`
- `ELIMINACION_USUARIO`

### UserStatus

Estados disponibles en la enumeracion:

- `Asset`
- `Idle`
- `Blocked`

Actualmente el flujo principal usa `Asset` y `Blocked` para permitir o denegar autenticacion.

## Configuracion

La aplicacion corre por defecto en el puerto `8081`.

Variables de entorno requeridas:

- `DB_URL`: URL JDBC de PostgreSQL.
- `DB_USERNAME`: usuario de base de datos.
- `DB_PASSWORD`: contrasena de base de datos.
- `JWT_SECRET`: secreto para firmar los tokens JWT.

Configuracion relevante en `application.yaml`:

- `spring.jpa.hibernate.ddl-auto=update`
- zona horaria `UTC` para JPA y Jackson
- exposicion de `health` e `info` en Actuator

## Ejecucion local

### Requisitos

- Java 17
- PostgreSQL disponible
- Variables de entorno configuradas

### PowerShell

```powershell
$env:DB_URL="jdbc:postgresql://localhost:5432/authdb"
$env:DB_USERNAME="postgres"
$env:DB_PASSWORD="tu_clave"
$env:JWT_SECRET="tu_clave_secreta"
cd auth-service
.\mvnw.cmd spring-boot:run
```

### Linux o macOS

```bash
export DB_URL=jdbc:postgresql://localhost:5432/authdb
export DB_USERNAME=postgres
export DB_PASSWORD=tu_clave
export JWT_SECRET=tu_clave_secreta
cd auth-service
./mvnw spring-boot:run
```

## Docker

Desde la carpeta `auth-service`:

```bash
docker build -t auth-service .
docker run -p 8081:8081 --env-file .env auth-service
```

## Endpoints principales

Base URL local: `http://localhost:8081`

### Autenticacion

#### `POST /api/auth/login`

Autentica un usuario y retorna un token JWT.

Solicitud:

```json
{
  "email": "admin@farmaexpres.com",
  "password": "ClaveSegura123"
}
```

Respuesta:

```json
{
  "token": "jwt-token",
  "type": "Bearer",
  "email": "admin@farmaexpres.com",
  "role": "ADMIN",
  "name": "Juan Perez"
}
```

### Usuarios

#### `POST /api/users`

Crea un nuevo usuario.

```json
{
  "name": "Juan Perez",
  "email": "juan@farmaexpres.com",
  "password": "ClaveSegura123",
  "role": "ADMIN"
}
```

#### `GET /api/users`

Lista los usuarios registrados.

```json
[
  {
    "id": 1,
    "name": "Juan Perez",
    "email": "juan@farmaexpres.com",
    "role": "ADMIN",
    "state": "Asset"
  }
]
```

#### `PUT /api/users/{id}/update`

Actualiza nombre, correo y/o rol. La actualizacion es parcial segun los campos enviados.

```json
{
  "name": "Juan Actualizado",
  "email": "juan.actualizado@farmaexpres.com",
  "role": "AUDITOR"
}
```

#### `PUT /api/users/{id}/password`

Cambia la contrasena del usuario.

```json
{
  "currentpassword": "ClaveActual123",
  "newPassword": "ClaveNueva123"
}
```

Respuesta exitosa:

```text
Contrasena actualizada correctamente
```

#### `PUT /api/users/{id}/block`

Bloquea una cuenta.

#### `PUT /api/users/{id}/unlock`

Desbloquea una cuenta.

#### `DELETE /api/users/{id}`

Elimina un usuario.

### Bitacora

#### `GET /api/binnacle`

Devuelve la lista completa de eventos registrados.

### Estado del servicio

#### `GET /status`

Respuesta:

```json
{
  "status": "ok"
}
```

#### `GET /actuator/health`

Chequeo de salud del servicio.

#### `GET /actuator/info`

Informacion general expuesta por Actuator.

## Seguridad

El proyecto usa JWT y Spring Security. Las peticiones protegidas deben incluir:

```http
Authorization: Bearer <token>
```

Endpoints publicos:

- `POST /api/auth/login`
- `GET /status`
- `GET /actuator/health`
- `GET /actuator/info`

Permisos configurados actualmente:

- `ADMIN`: acceso total a `/api/users/**`, consulta de usuarios, bitacora y cambio de contrasena.
- `AUDITOR`: puede consultar usuarios, consultar bitacora, cambiar contrasena y actualizar usuarios por la regla actual de seguridad.
- `FARMACEUTICO`: puede cambiar contrasena y actualizar usuarios por la regla actual de seguridad.

Nota: la ruta `PUT /api/users/{id}/update` no esta restringida solo a administradores en la configuracion actual; tambien acepta `AUDITOR` y `FARMACEUTICO`.

## Validaciones implementadas

### Correo

Se valida de forma explicita que:

- no sea nulo ni vacio
- no contenga espacios
- tenga una sola arroba
- la parte local y el dominio tengan formato valido
- no use dominios internacionalizados en punycode
- el TLD tenga al menos 2 letras

### Nombre

Se valida y normaliza para que:

- no sea nulo ni vacio
- elimine espacios sobrantes
- solo contenga letras y espacios

### Cambio de contrasena

Reglas aplicadas:

- la contrasena actual es obligatoria
- la nueva contrasena es obligatoria
- la nueva contrasena debe tener minimo 8 caracteres
- la nueva contrasena debe ser diferente a la actual

## Manejo de errores

Las excepciones se unifican con `GlobalExceptionHandler` y responden en formato JSON:

```json
{
  "timestamp": "2026-04-05T19:20:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "El correo es obligatorio.",
  "path": "/api/auth/login",
  "service": "auth-service"
}
```

Codigos frecuentes:

- `400 Bad Request`: errores de validacion o reglas de negocio.
- `401 Unauthorized`: credenciales invalidas.
- `403 Forbidden`: usuario sin acceso o cuenta no activa.
- `404 Not Found`: recurso o rol inexistente.
- `409 Conflict`: correo ya registrado.
- `500 Internal Server Error`: error inesperado.

## Pruebas

Para ejecutar las pruebas:

```powershell
cd auth-service
.\mvnw.cmd test
```

En Linux o macOS:

```bash
cd auth-service
./mvnw test
```
