# 🚀 Load Balance Taller - Microservicios con JWT Security

> Implementación de una arquitectura de microservicios con Spring Boot, Eureka Server, Load Balancer y autenticación JWT utilizando el patrón Access Token.

## Descripción del Proyecto

Este proyecto implementa una arquitectura de microservicios completa que incluye:

- **Eureka Server**: Registro y descubrimiento de servicios
- **Eureka Client**: Servicio proveedor de mensajes
- **Load Balancer**: Servicio consumidor con balanceador de carga y seguridad JWT
- **JWT Security**: Implementación del patrón Access Token para proteger endpoints

## Arquitectura del Sistema

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Eureka Server │    │  Eureka Client  │    │  Load Balancer  │
│   (Puerto 8761) │    │  (Puerto 8080)  │    │  (Puerto 8084)  │
│                 │    │                 │    │                 │
│  - Registro de  │◄──►│  - Servicio     │◄──►│  - JWT Security │
│    servicios    │    │    /message     │    │  - Endpoint     │
│  - Dashboard    │    │  - Proveedor    │    │    protegido    │
│                 │    │                 │    │  - Load Balance │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Tecnologías Utilizadas

- **Java 17**
- **Spring Boot 3.5.5**
- **Spring Cloud 2025.0.0**
- **Spring Security**
- **Netflix Eureka**
- **JSON Web Tokens (JWT)**
- **Maven**

## Estructura del Proyecto

```
load_balance_taller/
├── eureka-server/          # Servidor de registro de servicios
│   ├── src/main/java/
│   │   └── co/edu/uptc/
│   │       └── EurekaServerApplication.java
│   ├── src/main/resources/
│   │   └── application.yml
│   └── pom.xml
├── eureka-client/          # Cliente proveedor de servicios
│   ├── src/main/java/
│   │   └── co/edu/uptc/
│   │       ├── EurekaClientApplication.java
│   │       └── controller/
│   │           └── MessageController.java
│   ├── src/main/resources/
│   │   └── application.yml
│   └── pom.xml
├── load_balancer/          # Load balancer con JWT Security
│   ├── src/main/java/
│   │   └── co/edu/uptc/
│   │       ├── LoadBalancerApplication.java
│   │       ├── config/
│   │       │   ├── AppConfig.java
│   │       │   └── SecurityConfig.java
│   │       ├── controller/
│   │       │   ├── AuthController.java
│   │       │   └── MessageController.java
│   │       ├── filter/
│   │       │   └── JwtAuthenticationFilter.java
│   │       ├── model/
│   │       │   ├── AuthRequest.java
│   │       │   └── AuthResponse.java
│   │       └── util/
│   │           └── JwtUtil.java
│   ├── src/main/resources/
│   │   └── application.yml
│   └── pom.xml
└── README.md
```

## Instalación y Configuración

### Prerrequisitos

- Java 17 o superior
- Maven 3.6 o superior
- Puerto 8761, 8080 (y otros para otras instancias) y 8084 disponibles

### Clonar el Repositorio

```bash
git clone https://github.com/la00429/load_balance_taller.git
cd load_balance_taller
```

### Ejecutar los Servicios

**IMPORTANTE: Ejecutar en este orden específico**

#### 1. Eureka Server (Puerto 8761)
```bash
cd eureka-server
mvn clean install
mvn spring-boot:run
```

#### 2. Eureka Client (Puerto 8080)
```bash
cd eureka-client
mvn clean install
mvn spring-boot:run
```
Lo anterior para 1 sola instancia, en otro caso, se debe ejecutar el jar que se encuentra en el directorio ./target/ dentro de la carpeta de eureka-client
```
java -jar .\target\eureka-client-0.0.1-SNAPSHOT.jar --server.port=808x

```
Recordar que el puerto 8084 y 8761 ya se encuentran en reserva. 
En otro caso, garantizar que los puertos están libres para ejecuarlos.

#### 3. Load Balancer (Puerto 8084)
```bash
cd load_balancer
mvn clean install
mvn spring-boot:run
```

### Verificar la Instalación

1. **Eureka Dashboard**: http://localhost:8761
2. **Health Check**: http://localhost:8084/health
3. **Cliente directo**: http://localhost:8080/message

Este último, puede preentar un error 403 denegación de acceso.

## Autenticación JWT

### Credenciales de Prueba

- **Usuario**: `admin`
- **Contraseña**: `password`

### Flujo de Autenticación

1. **Obtener Token JWT**
2. **Usar Token para Acceder a Endpoints Protegidos**

## API Endpoints

### Endpoints Públicos

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/auth/login` | Autenticación y obtención de token JWT |
| GET | `/auth/validate` | Validación de token JWT |
| GET | `/health` | Health check del servicio |
| GET | `/actuator/**` | Endpoints de monitoreo |

### Endpoints Protegidos

| Método | Endpoint | Descripción | Requiere JWT |
|--------|----------|-------------|--------------|
| GET | `/get-message` | Mensaje seguro con load balancing | Si |

## Ejemplos de Uso

### 1. Obtener Token JWT

**Request:**
```bash
curl -X POST http://localhost:8084/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}'
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImlhdCI6MTc1NzgyNjkyNywiZXhwIjoxNzU3OTEzMzI3fQ.QBguLK8tyd4l_sp1ZddJ0oJY4QvuDTW3t_zuK3QXjAU",
  "message": "Login exitoso"
}
```

### 2. Acceder a Endpoint Protegido

**Request:**
```bash
curl -X GET http://localhost:8084/get-message \
  -H "Authorization: Bearer [TOKEN_OBTENIDO]"
```
El bearer es porque así se indicó el formato en el controlador de auth
**Response:**
```
Mensaje seguro para usuario: admin | Hello from Eureka Client running on port: 8080
```

### 3. Probar sin Token (Error 403)

**Request:**
```bash
curl -X GET http://localhost:8084/get-message
```

**Response:**
```json
{
  "timestamp": "2025-09-14T05:09:52.226+00:00",
  "status": 403,
  "error": "Forbidden",
  "path": "/get-message"
}
```

## Pruebas con Postman

### 1. Login Request - Obtener Token JWT

**Configuración:**
- **Método**: `POST`
- **URL**: `http://localhost:8084/auth/login`

**Headers:**
- `Content-Type: application/json`

**Body:**
- Seleccionar **"raw"**
- En el dropdown seleccionar **"JSON"**
- Contenido:
```json
{
  "username": "admin",
  "password": "password"
}
```

**Respuesta esperada:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "message": "Login exitoso"
}
```

### 2. Endpoint Protegido - Get Message

**Configuración:**
- **Método**: `GET`
- **URL**: `http://localhost:8084/get-message`

**Headers:**
- `Authorization: Bearer [COPIAR_TOKEN_AQUI]`

**Ejemplo del header Authorization:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImlhdCI6MTc1NzgyNjkyNywiZXhwIjoxNzU3OTEzMzI3fQ.QBguLK8tyd4l_sp1ZddJ0oJY4QvuDTW3t_zuK3QXjAU
```

**Respuesta esperada:**
```
Mensaje seguro para usuario: admin | Hello from Eureka Client running on port: 8080
```

### 3. Validar Token (Opcional)

**Configuración:**
- **Método**: `GET`
- **URL**: `http://localhost:8084/auth/validate`

**Headers:**
- `Authorization: Bearer [TOKEN]`

**Respuesta esperada:**
```json
{
  "token": null,
  "message": "Token válido para usuario: admin"
}
```

### Notas Importantes para Postman:

1. **Copiar el token completo** de la respuesta del login
2. **Incluir "Bearer "** (con espacio) antes del token en el header Authorization
3. **No agregar comillas** alrededor del token
4. **Los tokens expiran en 24 horas** - generar uno nuevo si es necesario

## Configuración

### Puertos por Defecto

| Servicio | Puerto | Descripción |
|----------|--------|-------------|
| Eureka Server | 8761 | Registro de servicios |
| Eureka Client | 8080 | Servicio proveedor |
| Load Balancer | 8084 | Servicio consumidor con JWT |

### Configuración JWT

- **Algoritmo**: HS256
- **Expiración**: 24 horas
- **Clave secreta**: Configurable en `application.yml`

### Variables de Entorno

```yaml
jwt:
  secret: mySecretKey12345678901234567890123456789012345678901234567890
  expiration: 86400000 # 24 hours in milliseconds
```

## Características de Seguridad

- **Autenticación JWT** con tokens seguros
- **Validación automática** en cada request
- **Endpoints protegidos** con Spring Security
- **Sesiones stateless** (sin estado en servidor)
- **Encriptación BCrypt** para contraseñas
- **Headers Authorization** con formato Bearer

## Monitoreo y Logs

### Dashboard Eureka
Accede a http://localhost:8761 para ver:
- Servicios registrados
- Estado de instancias
- Información de salud

### Logs de Seguridad
Los logs incluyen información detallada sobre:
- Intentos de autenticación
- Validación de tokens
- Acceso a endpoints protegidos
 Revisar las terminales por el momento.
## Troubleshooting

### Problemas Comunes

1. **Puerto en uso**
   ```bash
   netstat -ano | findstr :8084
   taskkill /F /PID [PID_NUMBER]
   ```

2. **Token expirado**
   - Generar nuevo token con `/auth/login`
   - Los tokens expiran en 24 horas

3. **Servicios no se registran en Eureka**
   - Verificar que Eureka Server esté corriendo primero
   - Revisar configuración de `defaultZone` en `application.yml`

4. **Error 403 Forbidden**
   - Verificar que el token esté en el header `Authorization`
   - Formato correcto: `Bearer [TOKEN]`

### Logs Útiles

```bash
# Ver logs del Load Balancer
cd load_balancer
mvn spring-boot:run

# Buscar errores de autenticación
grep "Authentication failed" logs/application.log
```

## Contribuir

1. Fork el proyecto
2. Crear una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abrir un Pull Request

## Autores

- Cristian Basto, Katherine Bello y Laura Figueredo


⭐ **¡Si este proyecto te fue útil, no olvides darle una estrella!** ⭐
