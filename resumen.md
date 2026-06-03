# Vue 3 + TypeScript + Pinia + FastAPI + JWT

## Objetivo

Construir una aplicación frontend moderna utilizando:

- Vue 3
    
- TypeScript
    
- Pinia
    
- Vue Router
    
- Tailwind CSS
    

Conectada a una API desarrollada en FastAPI utilizando autenticación JWT.

---

# 1. Creación del proyecto

Se creó el proyecto utilizando Vite:

```bash
npm create vue@latest frontend-vue
```

Opciones seleccionadas:

```txt
TypeScript: Sí
Vue Router: Sí
Pinia: Sí
ESLint: Sí
Prettier: Sí
```

Instalación:

```bash
cd frontend-vue
npm install
npm run dev
```

---

# 2. Configuración de App.vue

Se simplificó App.vue para que únicamente renderice las rutas de Vue Router.

```vue
<template>
  <RouterView />
</template>
```

---

# 3. Estructura de carpetas

Se organizó el proyecto utilizando una estructura escalable.

```txt
src/
├── api/
├── components/
├── config/
├── interfaces/
├── layouts/
├── router/
├── stores/
└── views/
```

Descripción:

|Carpeta|Responsabilidad|
|---|---|
|api|Comunicación con FastAPI|
|components|Componentes reutilizables|
|config|Configuración global|
|interfaces|Tipado TypeScript|
|layouts|Diseños compartidos|
|router|Configuración de rutas|
|stores|Estado global con Pinia|
|views|Pantallas de la aplicación|

---

# 4. Interfaces TypeScript

Se crearon interfaces para tipar correctamente los datos.

## Usuario

```ts
export interface Usuario {
  id: number;
  nombre: string;
  correo: string;
}
```

## LoginRequest

```ts
export interface LoginRequest {
  correo: string;
  password: string;
}
```

## LoginResponse

```ts
export interface LoginResponse {
  access_token: string;
  token_type: string;
  usuario_id: number;
  nombre: string;
  correo: string;
}
```

## Producto

```ts
export interface Producto {
  id: number;
  nombre: string;
  descripcion: string | null;
  precio: number;
  stock: number;
  usuario_id: number;
}
```

---

# 5. Configuración de la API

Se centralizó la URL de FastAPI.

Archivo:

```txt
src/config/api.ts
```

```ts
export const API_URL = 'http://localhost:8000';
```

---

# 6. Capa API

Se crearon archivos responsables de comunicarse con FastAPI.

```txt
src/api/auth.api.ts
src/api/producto.api.ts
```

Responsabilidades:

## auth.api.ts

- Login
    

## producto.api.ts

- Listar productos
    
- Crear producto
    
- Actualizar producto
    
- Eliminar producto
    

---

# 7. Gestión de estado con Pinia

Archivo:

```txt
src/stores/auth.store.ts
```

Responsabilidades:

- Mantener usuario autenticado
    
- Mantener token JWT
    
- Gestionar login
    
- Gestionar logout
    
- Persistir sesión en localStorage
    

Estado:

```ts
token
usuario
```

Acciones:

```ts
login()
logout()
```

Getter:

```ts
estaAutenticado
```

---

# 8. Pantalla Login

Archivo:

```txt
src/views/LoginView.vue
```

Responsabilidades:

- Solicitar correo
    
- Solicitar contraseña
    
- Ejecutar login
    
- Guardar sesión
    
- Redirigir al dashboard
    

Flujo:

```txt
Usuario
↓
LoginView
↓
Pinia Store
↓
FastAPI
↓
JWT
↓
Dashboard
```

---

# 9. Configuración de rutas

Archivo:

```txt
src/router/index.ts
```

Rutas configuradas:

```txt
/login
/app
/app/productos
```

---

# 10. Protección de rutas

Se implementó un Navigation Guard.

```ts
router.beforeEach(...)
```

Responsabilidad:

```txt
Si el usuario no está autenticado
↓
Redirigir a /login
```

Protección aplicada a:

```txt
/app
/app/productos
```

---

# 11. Dashboard

Archivo:

```txt
src/views/DashboardView.vue
```

Responsabilidades:

- Mostrar bienvenida
    
- Mostrar usuario autenticado
    
- Navegar al mantenedor
    

---

# 12. Navbar

Archivo:

```txt
src/components/AppNavbar.vue
```

Funcionalidades:

- Nombre del sistema
    
- Navegación
    
- Usuario autenticado
    
- Logout
    

---

# 13. Layout Privado

Archivo:

```txt
src/layouts/PrivateLayout.vue
```

Objetivo:

Evitar repetir componentes comunes en cada pantalla.

Estructura:

```txt
Navbar
↓
RouterView
```

Todas las pantallas privadas utilizan este layout.

---

# 14. Mantenedor de Productos

Archivo:

```txt
src/views/ProductoView.vue
```

Funcionalidades:

## Listar productos

```txt
GET /productos
```

## Crear producto

```txt
POST /productos
```

## Actualizar producto

```txt
PUT /productos/{id}
```

## Eliminar producto

```txt
DELETE /productos/{id}
```

---

# 15. Instalación de Tailwind CSS

Instalación:

```bash
npm install tailwindcss @tailwindcss/vite
```

Configuración en:

```txt
vite.config.ts
```

Y:

```txt
src/assets/main.css
```

```css
@import "tailwindcss";
```

---

# 16. Aplicación de Tailwind

Se mejoró visualmente:

- Login
    
- Navbar
    
- Dashboard
    
- Layout
    
- Tabla de productos
    
- Formulario de productos
    
- Botones
    
- Mensajes de error
    

Beneficios:

```txt
Menos CSS manual
Diseño consistente
Desarrollo más rápido
Mejor mantenibilidad
```

---

# Arquitectura Final

```txt
Vue Router
      │
      ▼
PrivateLayout
      │
      ▼
Views
      │
      ▼
Pinia Store
      │
      ▼
API Layer
      │
      ▼
FastAPI
      │
      ▼
MySQL
```

---

# Flujo de Autenticación

```txt
Usuario
↓
LoginView
↓
Auth Store
↓
POST /auth/login
↓
FastAPI
↓
JWT
↓
Pinia + localStorage
↓
Dashboard
↓
Rutas protegidas
```

---

# Tecnologías Utilizadas

Frontend:

- Vue 3
    
- TypeScript
    
- Pinia
    
- Vue Router
    
- Tailwind CSS
    
- Vite
    

Backend:

- FastAPI
    
- SQLAlchemy
    
- MySQL
    
- JWT
    
- BCrypt
