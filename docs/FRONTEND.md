# Documentación del Frontend

Esta documentación detalla cada archivo del frontend React, explicando su propósito y funcionamiento.

---

## Índice de Archivos

1. [main.jsx - Punto de Entrada](#mainjsx---punto-de-entrada)
2. [App.jsx - Componente Principal](#appjsx---componente-principal)
3. [store/authStore.js - Estado Global](#storeauthstorejs---estado-global)
4. [Páginas](#páginas)
5. [Componentes](#componentes)
6. [Utilidades](#utilidades)

---

## main.jsx - Punto de Entrada

**Ubicación:** `frontend/src/main.jsx`

Archivo de entrada que inicializa la aplicación React.

### Estructura

```jsx
import './index.css'              // Estilos globales (Tailwind)
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import { BrowserRouter } from 'react-router-dom' 

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

### Componentes Clave

| Componente | Propósito |
|------------|-----------|
| `React.StrictMode` | Activa verificaciones adicionales en desarrollo |
| `BrowserRouter` | Habilita el enrutamiento SPA con React Router |

---

## App.jsx - Componente Principal

**Ubicación:** `frontend/src/App.jsx`

Componente raíz que define la estructura de rutas y protección de navegación.

### Componentes de Protección de Rutas

#### ProtectedRoute

```jsx
const ProtectedRoute = ({ children }) => {
  const { isAuthenticated, user } = useAuthStore();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if(user && !user.isVerified){
    return <Navigate to="/verify-email" replace />;
  }
  return children;
}
```

**Propósito:** Proteger rutas que requieren autenticación.

**Lógica:**
1. Si no está autenticado → redirige a `/login`
2. Si está autenticado pero email no verificado → redirige a `/verify-email`
3. Si cumple ambas condiciones → renderiza los hijos

---

#### RedirectAuthenticatedUser

```jsx
const RedirectAuthenticatedUser = ({ children }) => {
  const { isAuthenticated, user } = useAuthStore();
  if (isAuthenticated && user && user.isVerified) {
    return <Navigate to="/" replace />;
  }
  return children;
}
```

**Propósito:** Evitar que usuarios ya autenticados accedan a páginas de login/registro.

**Lógica:**
- Si está autenticado Y verificado → redirige al dashboard (`/`)
- Si no → renderiza los hijos (página de login/signup)

---

### Verificación de Autenticación al Cargar

```jsx
const {isCheckingAuth, checkAuth} = useAuthStore();

useEffect(() => {
  checkAuth();
}, [checkAuth]);

if(isCheckingAuth) return <LoadingSpinner/>; 
```

**Propósito:** Al cargar la app, verificar si existe una sesión activa.

**Flujo:**
1. Llama a `checkAuth()` que hace petición a `/api/auth/check-auth`
2. Mientras verifica, muestra el `LoadingSpinner`
3. Al completar, el estado `isCheckingAuth` cambia a `false`

---

### Estructura de Rutas

| Ruta | Componente | Protección |
|------|------------|------------|
| `/` | DashboardPage | ProtectedRoute |
| `/signup` | SignUpPage | RedirectAuthenticatedUser |
| `/login` | LoginPage | RedirectAuthenticatedUser |
| `/verify-email` | EmailVerificationPage | Ninguna |
| `/forgot-password` | ForgotPasswordPage | RedirectAuthenticatedUser |
| `/reset-password/:token` | ResetPasswordPage | RedirectAuthenticatedUser |

---

### Elementos Decorativos

```jsx
<FloatingShape color="bg-green-500" size="w-64 h-64" top="-5%" left="10%" delay={0} />
<FloatingShape color="bg-emerald-500" size="w-48 h-48" top="70%" left="80%" delay={5} />
<FloatingShape color="bg-lime-500" size="w-32 h-32" top="40%" left="-10%" delay={2} />
```

Tres formas animadas en diferentes posiciones y tamaños para decorar el fondo.

---

## store/authStore.js - Estado Global

**Ubicación:** `frontend/src/store/authStore.js`

Gestión de estado global usando **Zustand**, una alternativa ligera a Redux.

### Configuración de Axios

```javascript
const API_URL = import.meta.env.MODE === 'development' 
  ? 'http://localhost:8000/api/auth' 
  : '/api/auth';
axios.defaults.withCredentials = true;
```

- En desarrollo: Usa URL completa del backend (puerto 8000)
- En producción: Usa ruta relativa (mismo servidor)
- `withCredentials = true`: Envía cookies con cada petición

---

### Estado Inicial

| Propiedad | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `user` | Object/null | null | Datos del usuario actual |
| `isAuthenticated` | Boolean | false | Si hay sesión activa |
| `error` | String/null | null | Mensaje de error actual |
| `isLoading` | Boolean | false | Si hay petición en curso |
| `isCheckingAuth` | Boolean | true | Si está verificando sesión |
| `message` | String/null | null | Mensaje de éxito |

---

### Acciones

#### signUp(email, password, name)

**Endpoint:** `POST /api/auth/signup`

**Flujo:**
1. Activa `isLoading`, limpia errores
2. Envía petición POST con credenciales
3. Si éxito: guarda usuario, marca autenticado
4. Si error: guarda mensaje de error, lanza excepción

---

#### login(email, password)

**Endpoint:** `POST /api/auth/login`

**Flujo:**
1. Activa `isLoading`, limpia errores
2. Envía petición POST con credenciales
3. Si éxito: guarda usuario, marca autenticado
4. Si error: guarda mensaje de error, lanza excepción

---

#### logout()

**Endpoint:** `POST /api/auth/logout`

**Flujo:**
1. Activa `isLoading`
2. Envía petición POST
3. Limpia usuario, marca no autenticado

---

#### verifyEmail(code)

**Endpoint:** `POST /api/auth/verify-email`

**Flujo:**
1. Envía código de 6 dígitos
2. Si éxito: actualiza usuario con `isVerified: true`
3. Si error: guarda mensaje de error

---

#### checkAuth()

**Endpoint:** `GET /api/auth/check-auth`

**Flujo:**
1. Activa `isCheckingAuth`
2. Envía petición GET (con cookie JWT)
3. Si éxito: guarda usuario, marca autenticado
4. Si error: limpia estado (sesión expirada o inválida)

**Nota:** No lanza error porque es verificación silenciosa al cargar la app.

---

#### forgotPassword(email)

**Endpoint:** `POST /api/auth/forgot-password`

**Flujo:**
1. Envía email
2. Si éxito: guarda mensaje de confirmación
3. Si error: guarda mensaje de error

---

#### resetPassword(token, password)

**Endpoint:** `POST /api/auth/reset-password/{token}`

**Flujo:**
1. Envía token de URL y nueva contraseña
2. Si éxito: guarda mensaje de confirmación
3. Si error: guarda mensaje de error

---

## Páginas

### SignUpPage.jsx

**Ubicación:** `frontend/src/pages/SignUpPage.jsx`

Formulario de registro de nuevos usuarios.

**Estado Local:**
- `name`: Nombre del usuario
- `email`: Email del usuario
- `password`: Contraseña

**Características:**
- Campos con iconos (User, Mail, Lock)
- Medidor de fortaleza de contraseña
- Animación de entrada con Framer Motion
- Botón con loader durante envío
- Enlace a página de login

**Flujo al Enviar:**
1. Previene submit por defecto
2. Llama a `signUp()` del store
3. Si éxito: navega a `/verify-email`

---

### LoginPage.jsx

**Ubicación:** `frontend/src/pages/LoginPage.jsx`

Formulario de inicio de sesión.

**Estado Local:**
- `email`: Email del usuario
- `password`: Contraseña

**Características:**
- Campos de email y password con iconos
- Enlace a "Forgot Password"
- Mensaje de error si credenciales inválidas
- Enlace a página de registro

---

### EmailVerificationPage.jsx

**Ubicación:** `frontend/src/pages/EmailVerificationPage.jsx`

Página para ingresar el código de verificación de 6 dígitos.

**Estado Local:**
- `code`: Array de 6 elementos (un dígito por input)
- `inputRefs`: Referencias a los 6 inputs

**Características Especiales:**

1. **Auto-focus al siguiente campo:**
   ```jsx
   if (value && index < 5) {
     inputRefs.current[index + 1].focus();
   }
   ```

2. **Soporte para pegar código completo:**
   ```jsx
   if (value.length > 1) {
     const pastedCode = value.slice(0, 6).split("");
     // Distribuye en todos los campos
   }
   ```

3. **Retroceso inteligente:**
   ```jsx
   if (e.key === 'Backspace' && !code[index] && index > 0) {
     inputRefs.current[index - 1].focus();
   }
   ```

4. **Envío automático al completar:**
   ```jsx
   useEffect(() => {
     if (code.every((digit) => digit !== '')){
       handleSubmit(new Event('submit'));
     }
   },[code]);
   ```

---

### DashboardPage.jsx

**Ubicación:** `frontend/src/pages/DashboardPage.jsx`

Panel principal del usuario autenticado.

**Datos Mostrados:**
- Nombre del usuario
- Email del usuario
- Fecha de registro (formateada)
- Último inicio de sesión (formateada)

**Características:**
- Animaciones escalonadas (delay en cada sección)
- Botón de logout
- Estilos de glassmorphism

---

### ForgotPasswordPage.jsx

**Ubicación:** `frontend/src/pages/ForgotPasswordPage.jsx`

Formulario para solicitar reset de contraseña.

**Estado Local:**
- `email`: Email del usuario
- `isSubmitted`: Si ya se envió la solicitud

**Flujo:**
1. Usuario ingresa email
2. Al enviar, llama a `forgotPassword()`
3. Cambia vista a mensaje de confirmación
4. Muestra icono de email animado

---

### ResetPasswordPage.jsx

**Ubicación:** `frontend/src/pages/ResetPasswordPage.jsx`

Formulario para establecer nueva contraseña.

**Estado Local:**
- `password`: Nueva contraseña
- `confirmPassword`: Confirmación de contraseña

**Obtención del Token:**
```jsx
const { token } = useParams();
```
Extrae el token de la URL (`/reset-password/:token`).

**Validación:**
- Verifica que ambas contraseñas coincidan antes de enviar

**Flujo Post-Éxito:**
1. Muestra toast de éxito
2. Espera 2 segundos
3. Redirige a `/login`

---

## Componentes

### Input.jsx

**Ubicación:** `frontend/src/components/Input.jsx`

Componente de input reutilizable con icono.

**Props:**
| Prop | Tipo | Descripción |
|------|------|-------------|
| `icon` | Component | Componente de icono (Lucide) |
| `...props` | any | Props estándar de input (type, value, onChange, etc.) |

**Estructura:**
- Contenedor relativo
- Icono posicionado absolutamente a la izquierda
- Input con padding izquierdo para el icono

---

### FloatingShape.jsx

**Ubicación:** `frontend/src/components/FloatingShape.jsx`

Forma decorativa animada para el fondo.

**Props:**
| Prop | Tipo | Ejemplo | Descripción |
|------|------|---------|-------------|
| `color` | String | "bg-green-500" | Clase de color Tailwind |
| `size` | String | "w-64 h-64" | Clases de tamaño |
| `top` | String | "-5%" | Posición vertical |
| `left` | String | "10%" | Posición horizontal |
| `delay` | Number | 0 | Delay de animación (segundos) |

**Animación:**
- Movimiento vertical y horizontal infinito
- Rotación de 360°
- Duración: 20 segundos
- Easing: linear

**Estilos:**
- `opacity-20`: Semi-transparente
- `blur-xl`: Efecto difuminado
- `rounded-full`: Forma circular

---

### LoadingSpinner.jsx

**Ubicación:** `frontend/src/components/LoadingSpinner.jsx`

Spinner de carga a pantalla completa.

**Uso:** Se muestra mientras la app verifica si hay sesión activa (`isCheckingAuth`).

**Animación:**
- Borde circular que rota infinitamente
- Duración: 1 segundo por rotación

---

### PasswordStrengthMeter.jsx

**Ubicación:** `frontend/src/components/PasswordStrengthMeter.jsx`

Indicador visual de la fortaleza de la contraseña.

**Subcomponente: PasswordCriteria**

Muestra lista de criterios con ✓ o ✗:
- Mínimo 6 caracteres
- Contiene mayúscula
- Contiene minúscula
- Contiene número
- Contiene carácter especial

**Componente Principal:**

**Cálculo de Fortaleza:**
```jsx
const getStrength = (pass) => {
  let strength = 0;
  if (pass.length >= 6) strength++;
  if (pass.match(/[a-z]/) && pass.match(/[A-Z]/)) strength++;
  if (pass.match(/\d/)) strength++;
  if (pass.match(/[^a-zA-Z\d]/)) strength++;
  return strength;
};
```

**Niveles de Fortaleza:**
| Valor | Texto | Color |
|-------|-------|-------|
| 0 | Very Weak | Rojo |
| 1 | Weak | Rojo claro |
| 2 | Fair | Amarillo |
| 3 | Good | Amarillo claro |
| 4 | Strong | Verde |

**Visualización:**
- 4 barras que se llenan según la fortaleza
- Texto descriptivo del nivel
- Lista de criterios cumplidos/pendientes

---

## Utilidades

### date.js

**Ubicación:** `frontend/src/utils/date.js`

Función para formatear fechas.

```javascript
export const formatDate = (dateString) => {
  const date = new Date(dateString);
  if (isNaN(date.getTime())) {
    return "Invalid Date";
  }

  return date.toLocaleString("en-US", {
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    hour12: true,
  });
};
```

**Ejemplo de Salida:** "Dec 21, 2025, 10:30 AM"

**Manejo de Errores:** Si la fecha es inválida, retorna "Invalid Date".

---

## Estilos y Diseño

### Tema de Colores

El proyecto usa una paleta de verdes:
- **Primary:** green-500, emerald-500
- **Background:** gray-800, gray-900
- **Text:** white, gray-300, gray-400
- **Error:** red-500
- **Success:** green-500

### Efectos Visuales

- **Glassmorphism:** `bg-opacity-50 backdrop-filter backdrop-blur-xl`
- **Gradientes:** `bg-gradient-to-r from-green-400 to-emerald-500`
- **Sombras:** `shadow-xl`, `shadow-2xl`
- **Bordes redondeados:** `rounded-lg`, `rounded-2xl`

### Animaciones (Framer Motion)

| Tipo | Configuración |
|------|---------------|
| Entrada | `initial={{ opacity: 0, y: 20 }}` |
| Visible | `animate={{ opacity: 1, y: 0 }}` |
| Hover | `whileHover={{ scale: 1.02 }}` |
| Click | `whileTap={{ scale: 0.98 }}` |
| Duración | `transition={{ duration: 0.5 }}` |
