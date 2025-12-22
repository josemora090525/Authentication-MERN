# Sistema de Autenticación Full-Stack

Un sistema de autenticación completo construido con **Node.js/Express** en el backend y **React** en el frontend. Incluye registro de usuarios, verificación de email, login/logout, recuperación de contraseña y protección de rutas.

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)

## Tabla de Contenidos

- [Características](#-características)
- [Tecnologías Utilizadas](#-tecnologías-utilizadas)
- [Estructura del Proyecto](#-estructura-del-proyecto)
- [Instalación](#-instalación)
- [Variables de Entorno](#-variables-de-entorno)
- [Arquitectura del Backend](#-arquitectura-del-backend)
- [Arquitectura del Frontend](#-arquitectura-del-frontend)
- [API Endpoints](#-api-endpoints)
- [Flujo de Autenticación](#-flujo-de-autenticación)

---

## Características

- **Registro de usuarios** con validación de campos
- **Verificación de email** mediante código de 6 dígitos
- **Inicio de sesión** con credenciales encriptadas
- **Recuperación de contraseña** vía email
- **Tokens JWT** almacenados en cookies HTTP-only
- **Protección de rutas** tanto en backend como frontend
- **Medidor de fortaleza de contraseña** en tiempo real
- **Animaciones fluidas** con Framer Motion
- **Diseño responsivo** con Tailwind CSS

---

## Tecnologías Utilizadas

### Backend
| Tecnología | Propósito |
|------------|-----------|
| **Express.js** | Framework web para Node.js |
| **MongoDB + Mongoose** | Base de datos NoSQL y ODM |
| **JWT (jsonwebtoken)** | Autenticación basada en tokens |
| **bcryptjs** | Encriptación de contraseñas |
| **Mailtrap** | Servicio de envío de emails |
| **cookie-parser** | Manejo de cookies |
| **cors** | Habilitación de peticiones cross-origin |

### Frontend
| Tecnología | Propósito |
|------------|-----------|
| **React 18** | Librería de UI |
| **Vite** | Bundler y servidor de desarrollo |
| **React Router DOM** | Navegación SPA |
| **Zustand** | Gestión de estado global |
| **Axios** | Cliente HTTP |
| **Framer Motion** | Animaciones |
| **Tailwind CSS** | Framework de estilos |
| **Lucide React** | Iconos |
| **React Hot Toast** | Notificaciones |

---

## Estructura del Proyecto

```
Authentication/
├── package.json              # Configuración raíz del proyecto
├── backend/
│   ├── index.js              # Punto de entrada del servidor
│   ├── controllers/
│   │   └── auth.controller.js    # Lógica de autenticación
│   ├── db/
│   │   └── connectDB.js          # Conexión a MongoDB
│   ├── mailtrap/
│   │   ├── emails.js             # Funciones de envío de emails
│   │   ├── emailTemplates.js     # Plantillas HTML de emails
│   │   └── mailtrap.config.js    # Configuración de Mailtrap
│   ├── middleware/
│   │   └── verifyToken.js        # Middleware de verificación JWT
│   ├── models/
│   │   └── user.model.js         # Esquema de usuario Mongoose
│   ├── routes/
│   │   └── auth.route.js         # Definición de rutas API
│   └── utils/
│       └── generateTokenAndSetCookie.js  # Utilidad para JWT
│
└── frontend/
    ├── index.html
    ├── package.json
    ├── vite.config.js
    ├── tailwind.config.js
    └── src/
        ├── App.jsx               # Componente principal y rutas
        ├── main.jsx              # Punto de entrada React
        ├── index.css             # Estilos globales
        ├── components/
        │   ├── FloatingShape.jsx     # Formas animadas decorativas
        │   ├── Input.jsx             # Input reutilizable con icono
        │   ├── LoadingSpinner.jsx    # Spinner de carga
        │   └── PasswordStrengthMeter.jsx  # Medidor de contraseña
        ├── pages/
        │   ├── SignUpPage.jsx        # Página de registro
        │   ├── LoginPage.jsx         # Página de login
        │   ├── EmailVerificationPage.jsx  # Verificación de email
        │   ├── DashboardPage.jsx     # Dashboard del usuario
        │   ├── ForgotPasswordPage.jsx    # Solicitar reset
        │   └── ResetPasswordPage.jsx     # Establecer nueva contraseña
        ├── store/
        │   └── authStore.js          # Estado global con Zustand
        └── utils/
            └── date.js               # Formateo de fechas
```

---

## Instalación

### Prerrequisitos
- Node.js v18 o superior
- MongoDB (local o Atlas)
- Cuenta en Mailtrap para emails

### Pasos

1. **Clonar el repositorio**
```bash
git clone <url-del-repositorio>
cd Authentication
```

2. **Instalar dependencias**
```bash
# Instalar dependencias del backend (raíz)
npm install

# Instalar dependencias del frontend
cd frontend && npm install
```

3. **Configurar variables de entorno** (ver sección siguiente)

4. **Ejecutar en desarrollo**
```bash
# Desde la raíz del proyecto
npm run dev        # Inicia el backend en modo desarrollo

# En otra terminal, para el frontend
cd frontend && npm run dev
```

5. **Build para producción**
```bash
npm run build      # Construye el frontend
npm start          # Inicia en modo producción
```

---

## Variables de Entorno

Crear un archivo `.env` en la raíz del proyecto:

```env
# MongoDB
MONGO_URI=mongodb+srv://<usuario>:<password>@cluster.mongodb.net/auth_db

# JWT
JWT_SECRET=tu_clave_secreta_muy_segura

# Servidor
PORT=8000
NODE_ENV=development

# Mailtrap
MAILTRAP_TOKEN=tu_token_de_mailtrap
MAILTRAP_ENDPOINT=https://send.api.mailtrap.io/

# Cliente (para emails de reset)
CLIENT_URL=http://localhost:5173
```

---

## Arquitectura del Backend

### Punto de Entrada (`backend/index.js`)

El servidor Express configura:
- **CORS** para permitir peticiones desde el frontend (puerto 5173)
- **Parsers** para JSON y cookies
- **Rutas** bajo el prefijo `/api/auth`
- **Servir archivos estáticos** del frontend en producción

### Modelo de Usuario (`backend/models/user.model.js`)

```
User Schema
├── email          (String, único, requerido)
├── password       (String, requerido, hasheado)
├── name           (String, requerido)
├── lastLogin      (Date)
├── isVerified     (Boolean, default: false)
├── verificationToken         (String)
├── verificationTokenExpiresAt (Date)
├── resetPasswordToken        (String)
├── resetPasswordExpiresAt    (Date)
└── timestamps     (createdAt, updatedAt automáticos)
```

### Controlador de Autenticación (`backend/controllers/auth.controller.js`)

| Función | Descripción |
|---------|-------------|
| `signup` | Registra usuario, hashea contraseña, genera token de verificación, envía email |
| `verifyEmail` | Valida código de 6 dígitos y marca usuario como verificado |
| `login` | Valida credenciales, genera JWT, actualiza `lastLogin` |
| `logout` | Limpia la cookie del token |
| `forgotPassword` | Genera token de reset y envía email con enlace |
| `resetPassword` | Valida token y actualiza contraseña |
| `checkAuth` | Devuelve datos del usuario autenticado |

### Middleware de Verificación (`backend/middleware/verifyToken.js`)

Extrae el JWT de las cookies, lo verifica y adjunta `userId` al request para rutas protegidas.

### Sistema de Emails (`backend/mailtrap/`)

- **mailtrap.config.js**: Configura cliente de Mailtrap
- **emailTemplates.js**: Plantillas HTML para verificación, reset y confirmación
- **emails.js**: Funciones para enviar cada tipo de email

---

## Arquitectura del Frontend

### Gestión de Estado (`frontend/src/store/authStore.js`)

Utiliza **Zustand** para manejar el estado global de autenticación:

```javascript
Estado:
├── user              // Datos del usuario actual
├── isAuthenticated   // Boolean de sesión activa
├── isLoading         // Estado de carga de peticiones
├── isCheckingAuth    // Verificando sesión al cargar app
├── error             // Mensajes de error
└── message           // Mensajes de éxito

Acciones:
├── signUp()          // Registro de usuario
├── login()           // Inicio de sesión
├── logout()          // Cerrar sesión
├── verifyEmail()     // Verificar código de email
├── checkAuth()       // Verificar sesión activa
├── forgotPassword()  // Solicitar reset de contraseña
└── resetPassword()   // Establecer nueva contraseña
```

### Protección de Rutas (`frontend/src/App.jsx`)

```jsx
ProtectedRoute        // Requiere autenticación Y email verificado
RedirectAuthenticatedUser  // Redirige a dashboard si ya está autenticado
```

### Componentes Reutilizables

| Componente | Función |
|------------|---------|
| `Input` | Campo de entrada con icono integrado |
| `FloatingShape` | Formas animadas de fondo decorativas |
| `LoadingSpinner` | Indicador de carga mientras verifica auth |
| `PasswordStrengthMeter` | Barra visual + criterios de contraseña |

---

## API Endpoints

Todos los endpoints están bajo `/api/auth`

| Método | Ruta | Protegido | Descripción |
|--------|------|-----------|-------------|
| `POST` | `/signup` | No | Registrar nuevo usuario |
| `POST` | `/login` | No | Iniciar sesión |
| `POST` | `/logout` | No | Cerrar sesión |
| `POST` | `/verify-email` | No | Verificar email con código |
| `POST` | `/forgot-password` | No | Solicitar email de reset |
| `POST` | `/reset-password/:token` | No | Establecer nueva contraseña |
| `GET` | `/check-auth` | Sí | Verificar sesión activa |

---

## Flujo de Autenticación

### Registro de Usuario
```
1. Usuario completa formulario → POST /signup
2. Backend hashea contraseña con bcrypt
3. Genera código de verificación de 6 dígitos
4. Guarda usuario en MongoDB con isVerified: false
5. Envía email con código vía Mailtrap
6. Frontend redirige a página de verificación
```

### Verificación de Email
```
1. Usuario ingresa código de 6 dígitos → POST /verify-email
2. Backend busca usuario con código válido y no expirado
3. Marca isVerified: true, limpia tokens
4. Envía email de bienvenida
5. Frontend redirige al dashboard
```

### Login
```
1. Usuario ingresa credenciales → POST /login
2. Backend valida email existe y contraseña coincide
3. Genera JWT con userId, expira en 30 días
4. Establece cookie HTTP-only con el token
5. Frontend guarda datos de usuario en Zustand
```

### Recuperación de Contraseña
```
1. Usuario ingresa email → POST /forgot-password
2. Backend genera token hexadecimal aleatorio
3. Guarda token con expiración de 1 hora
4. Envía email con enlace: /reset-password/:token
5. Usuario hace clic → POST /reset-password/:token
6. Backend valida token, hashea nueva contraseña
7. Envía email de confirmación
```

---

## Seguridad Implementada

- Contraseñas hasheadas con **bcrypt** (factor 10)
- Tokens JWT con expiración de 30 días
- Cookies **HTTP-only** (previene XSS)
- Cookies **secure** en producción (HTTPS)
- Cookies **sameSite: strict** (previene CSRF)
- Tokens de verificación expiran en 24 horas
- Tokens de reset expiran en 1 hora
- Validación de campos en backend
- Contraseñas nunca enviadas al frontend

---

## Licencia

ISC

---

## Autor

Jose David

---

## Contribuciones

Las contribuciones son bienvenidas. Por favor, abre un issue primero para discutir los cambios propuestos.
