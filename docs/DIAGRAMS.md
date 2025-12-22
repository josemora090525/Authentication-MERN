# Diagramas y Flujos del Sistema

Este documento contiene diagramas visuales que explican los flujos principales del sistema de autenticación.

---

## Arquitectura General

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENTE (Browser)                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                         React Frontend (Vite)                           ││
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────────┐  ││
│  │  │   Páginas    │  │  Componentes │  │     Estado (Zustand)         │  ││
│  │  │  - SignUp    │  │  - Input     │  │  - user                      │  ││
│  │  │  - Login     │  │  - Spinner   │  │  - isAuthenticated           │  ││
│  │  │  - Dashboard │  │  - Password  │  │  - isLoading                 │  ││
│  │  │  - Verify    │  │    Meter     │  │  - error                     │  ││
│  │  └──────────────┘  └──────────────┘  └──────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                      │                                       │
│                              Axios + Cookies                                 │
└──────────────────────────────────────┼───────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SERVIDOR (Node.js/Express)                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                              Middlewares                                 ││
│  │           CORS → JSON Parser → Cookie Parser → Routes                   ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                      │                                       │
│  ┌───────────────┐  ┌────────────────┼───────────────┐  ┌─────────────────┐ │
│  │    Routes     │  │           Controllers          │  │    Middleware   │ │
│  │  /api/auth/*  │──│  signup, login, logout, etc.  │──│   verifyToken   │ │
│  └───────────────┘  └────────────────┼───────────────┘  └─────────────────┘ │
│                                      │                                       │
│  ┌───────────────┐  ┌────────────────┼───────────────┐  ┌─────────────────┐ │
│  │    Models     │  │             Utils              │  │    Mailtrap     │ │
│  │  User Schema  │  │   generateTokenAndSetCookie   │  │  Email Service  │ │
│  └───────────────┘  └────────────────────────────────┘  └─────────────────┘ │
└──────────────────────────────────────┼───────────────────────────────────────┘
                                       │
                                       ▼
                     ┌─────────────────────────────────────┐
                     │            MongoDB Atlas            │
                     │         (Base de Datos)             │
                     │  ┌─────────────────────────────┐   │
                     │  │      Colección: Users       │   │
                     │  │  - email, password, name    │   │
                     │  │  - isVerified, lastLogin    │   │
                     │  │  - tokens de verificación   │   │
                     │  └─────────────────────────────┘   │
                     └─────────────────────────────────────┘
```

---

## Flujo de Registro (Sign Up)

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│  Usuario │         │ Frontend │         │ Backend  │         │ Mailtrap │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │                    │
     │ 1. Completa form   │                    │                    │
     │───────────────────>│                    │                    │
     │                    │                    │                    │
     │                    │ 2. POST /signup    │                    │
     │                    │───────────────────>│                    │
     │                    │                    │                    │
     │                    │                    │ 3. Hashea password │
     │                    │                    │ 4. Genera código   │
     │                    │                    │ 5. Guarda en DB    │
     │                    │                    │                    │
     │                    │                    │ 6. Envía email     │
     │                    │                    │───────────────────>│
     │                    │                    │                    │
     │                    │ 7. JWT en cookie   │                    │
     │                    │<───────────────────│                    │
     │                    │                    │                    │
     │ 8. Redirect a      │                    │                    │
     │    /verify-email   │                    │                    │
     │<───────────────────│                    │                    │
     │                    │                    │                    │
     │ 9. Recibe email    │                    │                    │
     │    con código      │                    │                    │
     │<─────────────────────────────────────────────────────────────│
     │                    │                    │                    │
```

---

## Flujo de Verificación de Email

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│  Usuario │         │ Frontend │         │ Backend  │         │ MongoDB  │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │                    │
     │ 1. Ingresa código  │                    │                    │
     │    de 6 dígitos    │                    │                    │
     │───────────────────>│                    │                    │
     │                    │                    │                    │
     │                    │ 2. POST            │                    │
     │                    │    /verify-email   │                    │
     │                    │    {code: 123456}  │                    │
     │                    │───────────────────>│                    │
     │                    │                    │                    │
     │                    │                    │ 3. Busca usuario   │
     │                    │                    │    con código      │
     │                    │                    │───────────────────>│
     │                    │                    │                    │
     │                    │                    │ 4. Usuario         │
     │                    │                    │<───────────────────│
     │                    │                    │                    │
     │                    │                    │ 5. Valida código   │
     │                    │                    │    y expiración    │
     │                    │                    │                    │
     │                    │                    │ 6. Actualiza       │
     │                    │                    │    isVerified=true │
     │                    │                    │───────────────────>│
     │                    │                    │                    │
     │                    │ 7. {success: true} │                    │
     │                    │<───────────────────│                    │
     │                    │                    │                    │
     │ 8. Redirect a /    │                    │                    │
     │    (Dashboard)     │                    │                    │
     │<───────────────────│                    │                    │
     │                    │                    │                    │
```

---

## Flujo de Login

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│  Usuario │         │ Frontend │         │ Backend  │
└────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │
     │ 1. Email + Pass    │                    │
     │───────────────────>│                    │
     │                    │                    │
     │                    │ 2. POST /login     │
     │                    │───────────────────>│
     │                    │                    │
     │                    │                    │ 3. Busca usuario
     │                    │                    │ 4. Compara password
     │                    │                    │    con bcrypt.compare
     │                    │                    │
     │                    │    ┌───────────────┴───────────────┐
     │                    │    │                               │
     │                    │    ▼                               ▼
     │                    │ ┌─────────┐                 ┌─────────────┐
     │                    │ │ Válido  │                 │  Inválido   │
     │                    │ └────┬────┘                 └──────┬──────┘
     │                    │      │                             │
     │                    │      │ 5. Genera JWT               │
     │                    │      │ 6. Set cookie               │
     │                    │      │ 7. Update lastLogin         │
     │                    │      │                             │
     │                    │      ▼                             ▼
     │                    │ ┌─────────────────┐     ┌─────────────────┐
     │                    │ │ 200 + userData  │     │ 400 + error     │
     │                    │ └────────┬────────┘     └────────┬────────┘
     │                    │          │                       │
     │                    │<─────────┴───────────────────────┘
     │                    │
     │ 8. Dashboard o     │
     │    Mostrar error   │
     │<───────────────────│
     │                    │
```

---

## Flujo de Recuperación de Contraseña

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│  Usuario │         │ Frontend │         │ Backend  │         │ Mailtrap │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │                    │
     │ 1. Click "Forgot   │                    │                    │
     │    Password"       │                    │                    │
     │───────────────────>│                    │                    │
     │                    │                    │                    │
     │ 2. Ingresa email   │                    │                    │
     │───────────────────>│                    │                    │
     │                    │                    │                    │
     │                    │ 3. POST            │                    │
     │                    │    /forgot-password│                    │
     │                    │───────────────────>│                    │
     │                    │                    │                    │
     │                    │                    │ 4. Genera token    │
     │                    │                    │    crypto.random   │
     │                    │                    │                    │
     │                    │                    │ 5. Guarda token    │
     │                    │                    │    en usuario      │
     │                    │                    │                    │
     │                    │                    │ 6. Envía email     │
     │                    │                    │    con enlace      │
     │                    │                    │───────────────────>│
     │                    │                    │                    │
     │                    │ 7. {message: ...}  │                    │
     │                    │<───────────────────│                    │
     │                    │                    │                    │
     │ 8. Muestra         │                    │                    │
     │    confirmación    │                    │                    │
     │<───────────────────│                    │                    │
     │                    │                    │                    │
     │ 9. Recibe email    │                    │                    │
     │    con enlace      │                    │                    │
     │<─────────────────────────────────────────────────────────────│
     │                    │                    │                    │
```

---

## Flujo de Reset de Contraseña

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│  Usuario │         │ Frontend │         │ Backend  │
└────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │
     │ 1. Click enlace    │                    │
     │    del email       │                    │
     │───────────────────>│                    │
     │                    │                    │
     │                    │ 2. Navega a        │
     │                    │    /reset/:token   │
     │                    │                    │
     │ 3. Ingresa nueva   │                    │
     │    contraseña x2   │                    │
     │───────────────────>│                    │
     │                    │                    │
     │                    │ 4. Valida que      │
     │                    │    coincidan       │
     │                    │                    │
     │                    │ 5. POST            │
     │                    │    /reset/:token   │
     │                    │    {password}      │
     │                    │───────────────────>│
     │                    │                    │
     │                    │                    │ 6. Busca usuario
     │                    │                    │    con token
     │                    │                    │
     │                    │                    │ 7. Valida
     │                    │                    │    expiración
     │                    │                    │
     │                    │                    │ 8. Hashea nueva
     │                    │                    │    contraseña
     │                    │                    │
     │                    │                    │ 9. Actualiza y
     │                    │                    │    limpia tokens
     │                    │                    │
     │                    │ 10. {success}      │
     │                    │<───────────────────│
     │                    │                    │
     │ 11. Toast + Redirect│                   │
     │     a /login        │                   │
     │<────────────────────│                   │
     │                    │                    │
```

---

## Flujo de Verificación de Autenticación (checkAuth)

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Browser  │         │ Frontend │         │ Backend  │
└────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │
     │ 1. Carga página    │                    │
     │───────────────────>│                    │
     │                    │                    │
     │                    │ 2. useEffect →     │
     │                    │    checkAuth()     │
     │                    │                    │
     │ 3. Muestra         │                    │
     │    LoadingSpinner  │                    │
     │<───────────────────│                    │
     │                    │                    │
     │                    │ 4. GET /check-auth │
     │                    │    (con cookie)    │
     │                    │───────────────────>│
     │                    │                    │
     │                    │                    │ 5. verifyToken
     │                    │                    │    middleware
     │                    │                    │
     │    ┌───────────────┴────────────────────┴───────────────┐
     │    │                                                     │
     │    ▼                                                     ▼
     │ ┌────────────────────┐                    ┌────────────────────┐
     │ │   Cookie válida    │                    │  Cookie inválida   │
     │ │   (JWT verificado) │                    │  o expirada        │
     │ └─────────┬──────────┘                    └─────────┬──────────┘
     │           │                                         │
     │           │ 6. Busca usuario                        │
     │           │    por userId                           │
     │           │                                         │
     │           ▼                                         ▼
     │ ┌────────────────────┐                    ┌────────────────────┐
     │ │ 200 + user data    │                    │ 401 Unauthorized   │
     │ └─────────┬──────────┘                    └─────────┬──────────┘
     │           │                                         │
     │<──────────┴─────────────────────────────────────────┘
     │
     │ 7. Actualiza estado:
     │    - isAuthenticated
     │    - user
     │    - isCheckingAuth = false
     │
     │ 8. Renderiza página
     │    según estado
     │
```

---

## Estructura del Token JWT

```
┌─────────────────────────────────────────────────────────────────────┐
│                           JSON Web Token                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Header (Base64)          Payload (Base64)         Signature        │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐│
│  │ {               │     │ {               │     │                 ││
│  │   "alg": "HS256"│  .  │   "userId": "..." │  .  │   HMACSHA256(   ││
│  │   "typ": "JWT"  │     │   "iat": 1234567│     │     base64(hdr) ││
│  │ }               │     │   "exp": 1237567│     │     base64(pay) ││
│  │                 │     │ }               │     │     secret      ││
│  │                 │     │                 │     │   )             ││
│  └─────────────────┘     └─────────────────┘     └─────────────────┘│
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                     Almacenado en Cookie                            │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  Name: token                                                    ││
│  │  Value: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...                ││
│  │  HttpOnly: true     (No accesible desde JavaScript)             ││
│  │  Secure: true       (Solo HTTPS en producción)                  ││
│  │  SameSite: Strict   (No enviada en requests cross-origin)       ││
│  │  MaxAge: 7 días                                                 ││
│  └─────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────┘
```

---

## Tipos de Emails Enviados

```
┌─────────────────────────────────────────────────────────────────────┐
│                       SISTEMA DE EMAILS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  1. VERIFICACIÓN DE EMAIL                                           │
│     ┌───────────────────────────────────────┐                      │
│     │  Subject: Email Verification        │                      │
│     │  Contenido: Código de 6 dígitos     │                      │
│     │  Expira: 24 horas                   │                      │
│     │  Trigger: Después de signup         │                      │
│     └───────────────────────────────────────┘                      │
│                                                                     │
│  2. BIENVENIDA                                                      │
│     ┌───────────────────────────────────────┐                      │
│     │  Subject: Welcome!                  │                      │
│     │  Template: Mailtrap UUID            │                      │
│     │  Trigger: Después de verificar      │                      │
│     └───────────────────────────────────────┘                      │
│                                                                     │
│  3. SOLICITUD DE RESET                                              │
│     ┌───────────────────────────────────────┐                      │
│     │  Subject: Reset your password          │                      │
│     │  Contenido: Enlace con token           │                      │
│     │  Expira: 1 hora                        │                      │
│     │  Trigger: Forgot password              │                      │
│     └───────────────────────────────────────┘                      │
│                                                                     │
│  4. CONFIRMACIÓN DE RESET                                           │
│     ┌───────────────────────────────────────┐                      │
│     │  Subject: Password Reset Successful    │                      │
│     │  Contenido: Confirmación + tips        │                      │
│     │  Trigger: Después de reset             │                      │
│     └───────────────────────────────────────┘                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Modelo de Datos en MongoDB

```
┌─────────────────────────────────────────────────────────────────────┐
│                         COLECCIÓN: users                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  {                                                                  │
│    "_id": ObjectId("..."),           // ID único de MongoDB         │
│                                                                     │
│    // ─────────── Datos Básicos ───────────                         │
│    "email": "user@example.com",      // Único, requerido            │
│    "password": "$2a$10$...",         // Hasheado con bcrypt         │
│    "name": "John Doe",               // Requerido                   │
│                                                                     │
│    // ─────────── Estado de Sesión ───────────                      │
│    "lastLogin": ISODate("..."),      // Última conexión             │
│    "isVerified": true,               // Email verificado            │
│                                                                     │
│    // ─────────── Tokens Temporales ───────────                     │
│    "verificationToken": "123456",    // 6 dígitos                   │
│    "verificationTokenExpiresAt": ISODate("..."),  // +24h           │
│                                                                     │
│    "resetPasswordToken": "a1b2c3...",  // 40 chars hex              │
│    "resetPasswordExpiresAt": ISODate("..."),      // +1h            │
│                                                                     │
│    // ─────────── Timestamps Automáticos ───────────                │
│    "createdAt": ISODate("..."),      // Fecha de registro           │
│    "updatedAt": ISODate("...")       // Última modificación         │
│  }                                                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```
