# Documentación del Backend

Esta documentación detalla cada archivo del backend, explicando su propósito y funcionamiento.

---

## Índice de Archivos

1. [index.js - Servidor Principal](#indexjs---servidor-principal)
2. [db/connectDB.js - Conexión a MongoDB](#dbconnectdbjs---conexión-a-mongodb)
3. [models/user.model.js - Modelo de Usuario](#modelsusermodeljs---modelo-de-usuario)
4. [routes/auth.route.js - Rutas de API](#routesauthroutejs---rutas-de-api)
5. [controllers/auth.controller.js - Controladores](#controllersauthcontrollerjs---controladores)
6. [middleware/verifyToken.js - Middleware JWT](#middlewareverifytokenjs---middleware-jwt)
7. [utils/generateTokenAndSetCookie.js - Utilidad JWT](#utilsgeneratetokenandsetcookiejs---utilidad-jwt)
8. [mailtrap/ - Sistema de Emails](#mailtrap---sistema-de-emails)

---

## index.js - Servidor Principal

**Ubicación:** `backend/index.js`

Este archivo es el punto de entrada de la aplicación. Configura y arranca el servidor Express.

### Importaciones

```javascript
import express from 'express';      // Framework web
import dotenv from 'dotenv';        // Variables de entorno desde .env
import cors from 'cors';            // Cross-Origin Resource Sharing
import cookieParser from 'cookie-parser';  // Parser de cookies
import path from 'path';            // Utilidades de rutas de archivos
import { connectDB } from './db/connectDB.js';  // Función de conexión a DB
import authRoutes from './routes/auth.route.js'; // Rutas de autenticación
```

### Configuración del Servidor

| Configuración | Descripción |
|---------------|-------------|
| `dotenv.config()` | Carga variables del archivo `.env` a `process.env` |
| `PORT` | Puerto del servidor (default: 8000) |
| `__dirname` | Directorio actual del proyecto |

### Middlewares

| Middleware | Propósito |
|------------|-----------|
| `cors({origin, credentials})` | Permite peticiones desde `localhost:5173` con cookies |
| `express.json()` | Parsea body de peticiones como JSON (`req.body`) |
| `cookieParser()` | Parsea cookies de las peticiones (`req.cookies`) |

### Rutas

- **`/api/auth`**: Prefijo para todas las rutas de autenticación

### Modo Producción

En producción (`NODE_ENV=production`):
- Sirve archivos estáticos del frontend (`frontend/dist`)
- Cualquier ruta no-API devuelve `index.html` (SPA)

### Inicio del Servidor

Al arrancar, se conecta a MongoDB y escucha en el puerto configurado.

---

## db/connectDB.js - Conexión a MongoDB

**Ubicación:** `backend/db/connectDB.js`

Establece la conexión con la base de datos MongoDB usando Mongoose.

### Funcionamiento

1. Lee la URI de MongoDB desde `process.env.MONGO_URI`
2. Intenta conectar usando `mongoose.connect()`
3. Si tiene éxito, muestra el host de conexión
4. Si falla, termina el proceso con código de error (1)

### Códigos de Salida

| Código | Significado |
|--------|-------------|
| 0 | Éxito |
| 1 | Error/Fallo |

---

## models/user.model.js - Modelo de Usuario

**Ubicación:** `backend/models/user.model.js`

Define la estructura de datos del usuario en MongoDB usando Mongoose Schema.

### Campos del Schema

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `email` | String | ✅ | Email único del usuario |
| `password` | String | ✅ | Contraseña hasheada con bcrypt |
| `name` | String | ✅ | Nombre completo |
| `lastLogin` | Date | ❌ | Fecha del último inicio de sesión |
| `isVerified` | Boolean | ❌ | Si el email está verificado (default: false) |
| `resetPasswordToken` | String | ❌ | Token para restablecer contraseña |
| `resetPasswordExpiresAt` | Date | ❌ | Expiración del token de reset |
| `verificationToken` | String | ❌ | Código de verificación de email |
| `verificationTokenExpiresAt` | Date | ❌ | Expiración del código de verificación |

### Opciones del Schema

- **`timestamps: true`**: Agrega automáticamente `createdAt` y `updatedAt`

---

## routes/auth.route.js - Rutas de API

**Ubicación:** `backend/routes/auth.route.js`

Define los endpoints de la API de autenticación.

### Endpoints Definidos

| Método | Ruta | Middleware | Controlador | Descripción |
|--------|------|------------|-------------|-------------|
| GET | `/check-auth` | `verifyToken` | `checkAuth` | Verifica sesión activa |
| POST | `/signup` | - | `signup` | Registro de usuario |
| POST | `/login` | - | `login` | Inicio de sesión |
| POST | `/logout` | - | `logout` | Cierre de sesión |
| POST | `/verify-email` | - | `verifyEmail` | Verificar código de email |
| POST | `/forgot-password` | - | `forgotPassword` | Solicitar reset |
| POST | `/reset-password/:token` | - | `resetPassword` | Nueva contraseña |

### Ruta Protegida

`/check-auth` usa el middleware `verifyToken` que valida el JWT antes de ejecutar el controlador.

---

## controllers/auth.controller.js - Controladores

**Ubicación:** `backend/controllers/auth.controller.js`

Contiene toda la lógica de negocio para las operaciones de autenticación.

### Dependencias

```javascript
import bcrypt from 'bcryptjs';    // Encriptación de contraseñas
import crypto from 'crypto';       // Generación de tokens aleatorios
import { generateTokenAndSetCookie } from '../utils/generateTokenAndSetCookie.js';
import { sendVerificationEmail, sendWelcomeEmail, ... } from "../mailtrap/emails.js";
import { User } from '../models/user.model.js';
```

---

### signup

**Propósito:** Registrar un nuevo usuario en el sistema.

**Flujo:**
1. Extrae `email`, `password`, `name` del body
2. Valida que todos los campos existan
3. Verifica que el email no esté registrado
4. Hashea la contraseña con bcrypt (factor 10)
5. Genera código de verificación de 6 dígitos
6. Crea usuario en la base de datos
7. Genera JWT y lo establece como cookie
8. Envía email de verificación
9. Responde con datos del usuario (sin contraseña)

**Código de verificación:** Número aleatorio entre 100000 y 999999

**Expiración:** 24 horas desde el registro

---

### verifyEmail

**Propósito:** Verificar el email del usuario con el código recibido.

**Flujo:**
1. Recibe el código de 6 dígitos
2. Busca usuario con ese código Y que no esté expirado
3. Marca `isVerified = true`
4. Limpia tokens de verificación
5. Envía email de bienvenida
6. Responde con datos del usuario

**Query MongoDB:** `verificationTokenExpiresAt: { $gt: Date.now() }` (mayor que fecha actual)

---

### login

**Propósito:** Autenticar usuario existente.

**Flujo:**
1. Recibe email y password
2. Busca usuario por email
3. Compara password con hash usando `bcrypt.compare()`
4. Genera nuevo JWT
5. Actualiza `lastLogin` con fecha actual
6. Responde con datos del usuario

**Seguridad:** Mensaje genérico "Invalid credentials" para email o password incorrectos (evita enumeration attacks)

---

### logout

**Propósito:** Cerrar la sesión del usuario.

**Flujo:**
1. Limpia la cookie "token"
2. Responde con mensaje de éxito

---

### forgotPassword

**Propósito:** Iniciar proceso de recuperación de contraseña.

**Flujo:**
1. Recibe email
2. Busca usuario
3. Genera token hexadecimal de 20 bytes (`crypto.randomBytes`)
4. Establece expiración de 1 hora
5. Guarda token en el usuario
6. Envía email con enlace: `{CLIENT_URL}/reset-password/{token}`

---

### resetPassword

**Propósito:** Establecer nueva contraseña.

**Flujo:**
1. Recibe token de URL params y nueva password del body
2. Busca usuario con token válido y no expirado
3. Hashea nueva contraseña
4. Actualiza password y limpia tokens
5. Envía email de confirmación

---

### checkAuth

**Propósito:** Verificar si el usuario tiene sesión activa.

**Flujo:**
1. El middleware `verifyToken` ya validó el JWT y agregó `userId`
2. Busca usuario por ID (excluyendo password)
3. Responde con datos del usuario

---

## middleware/verifyToken.js - Middleware JWT

**Ubicación:** `backend/middleware/verifyToken.js`

Middleware que protege rutas verificando el token JWT.

### Flujo de Verificación

1. Extrae token de `req.cookies.token`
2. Si no existe, responde 401 (Unauthorized)
3. Verifica token con `jwt.verify()` y `JWT_SECRET`
4. Si es inválido, responde 401
5. Extrae `userId` del token decodificado
6. Adjunta `userId` a `req.userId`
7. Llama a `next()` para continuar al controlador

### Respuestas de Error

| Código | Mensaje |
|--------|---------|
| 401 | "Unauthorized - no token provided" |
| 401 | "Unauthorized - invalid token" |
| 500 | "Server error" |

---

## utils/generateTokenAndSetCookie.js - Utilidad JWT

**Ubicación:** `backend/utils/generateTokenAndSetCookie.js`

Genera un JWT y lo establece como cookie HTTP-only.

### Parámetros

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `res` | Response | Objeto response de Express |
| `userId` | ObjectId | ID del usuario de MongoDB |

### Configuración del Token

| Propiedad | Valor | Descripción |
|-----------|-------|-------------|
| Payload | `{ userId }` | ID del usuario |
| Expiración | 30 días | Tiempo de vida del token |
| Secret | `JWT_SECRET` | Clave para firmar el token |

### Configuración de la Cookie

| Propiedad | Valor | Descripción |
|-----------|-------|-------------|
| `httpOnly` | true | No accesible desde JavaScript |
| `secure` | true en prod | Solo HTTPS en producción |
| `sameSite` | strict | Previene CSRF |
| `maxAge` | 7 días | Tiempo de vida de la cookie |

---

## mailtrap/ - Sistema de Emails

### mailtrap.config.js

**Ubicación:** `backend/mailtrap/mailtrap.config.js`

Configura el cliente de Mailtrap para envío de emails.

**Exportaciones:**
- `mailtrapClient`: Instancia configurada del cliente
- `sender`: Objeto con email y nombre del remitente

---

### emailTemplates.js

**Ubicación:** `backend/mailtrap/emailTemplates.js`

Contiene las plantillas HTML para los diferentes tipos de emails.

| Template | Propósito | Placeholder |
|----------|-----------|-------------|
| `VERIFICATION_EMAIL_TEMPLATE` | Verificación de email | `{verificationCode}` |
| `PASSWORD_RESET_REQUEST_TEMPLATE` | Solicitud de reset | `{resetURL}` |
| `PASSWORD_RESET_SUCCESS_TEMPLATE` | Confirmación de reset | - |

---

### emails.js

**Ubicación:** `backend/mailtrap/emails.js`

Funciones para enviar cada tipo de email.

| Función | Parámetros | Plantilla |
|---------|------------|-----------|
| `sendVerificationEmail` | email, verificationToken | VERIFICATION_EMAIL_TEMPLATE |
| `sendWelcomeEmail` | email, name | Template UUID de Mailtrap |
| `sendPasswordResetEmail` | email, resetURL | PASSWORD_RESET_REQUEST_TEMPLATE |
| `sendResetSuccessEmail` | email | PASSWORD_RESET_SUCCESS_TEMPLATE |

**Nota:** `sendWelcomeEmail` usa un template predefinido en Mailtrap (UUID) en lugar de HTML inline.

---

## Seguridad Implementada

### Encriptación de Contraseñas

```javascript
const hashedPassword = await bcrypt.hash(password, 10);
```
- Factor de costo: 10 (balance entre seguridad y rendimiento)
- Cada hash incluye salt único automáticamente

### Comparación Segura

```javascript
const isPasswordValid = await bcrypt.compare(password, user.password);
```
- Comparación en tiempo constante (previene timing attacks)

### Tokens de Verificación

- **Código de 6 dígitos:** Para verificación de email
- **Token hexadecimal de 40 caracteres:** Para reset de contraseña
- Ambos con expiración temporal

### Cookies Seguras

- **httpOnly:** Previene acceso desde JavaScript (XSS)
- **secure:** Solo enviadas por HTTPS (producción)
- **sameSite: strict:** Previene CSRF
