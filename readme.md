Seleccionar TYPESCRIPT si
- Router
- Pinia
- Linter
- Prettier
No elegir templates de ejemplo. Ingresar a la carpeta recién creada y ejecutar con pnpm run dev
Utiliza el puerto: http://localhost:5173

En la carpeta SRC deben quedar dentro las siguientes carpetas:
- api
- components
- interfaces
- router
- stores
- views

Y los archivos
- App.vue
- main.ts

Se crea en interfaces el archivo usuario.interface.ts
```
export interface Usuario {
  id: number;
  nombre: string;
  correo: string;
}
```

Se crea en interfaces el archivo auth.interface.ts
```
export interface LoginRequest {
  correo: string;
  password: string;
}
```

```
export interface LoginResponse {
  access_token: string;
  token_type: string;
  usuario_id: number;
  nombre: string;
  correo: string;
}
```

En api, crear el archivo auth.api.ts
```
const API_URL = 'http://localhost:8000';

  

import type {

LoginRequest,

LoginResponse

} from '@/interfaces/auth.interface';

  

export async function login(

request: LoginRequest

): Promise<LoginResponse> {

  

const response = await fetch(

`${API_URL}/auth/login`,

{

method: 'POST',

headers: {

'Content-Type': 'application/json'

},

body: JSON.stringify(request)

}

);

  

if (!response.ok) {

throw new Error('Credenciales incorrectas');

}

  

return await response.json();

}
```

En stores, crear el archivo auth.store.ts
```
import { defineStore } from 'pinia';

import type { Usuario } from '@/interfaces/usuario.interface';
import type { LoginRequest } from '@/interfaces/auth.interface';

import { login as loginApi } from '@/api/auth.api';

interface AuthState {
  token: string | null;
  usuario: Usuario | null;
}

export const useAuthStore = defineStore('auth', {
  state: (): AuthState => ({
    token: localStorage.getItem('token'),
    usuario: JSON.parse(localStorage.getItem('usuario') || 'null')
  }),

  getters: {
    estaAutenticado: (state) => !!state.token
  },

  actions: {
    async login(request: LoginRequest) {
      const data = await loginApi(request);

      this.token = data.access_token;

      this.usuario = {
        id: data.usuario_id,
        nombre: data.nombre,
        correo: data.correo
      };

      localStorage.setItem('token', this.token);
      localStorage.setItem('usuario', JSON.stringify(this.usuario));
    },

    logout() {
      this.token = null;
      this.usuario = null;

      localStorage.removeItem('token');
      localStorage.removeItem('usuario');
    }
  }
});
```

En views crear el archivo LoginView.vue
```
<script setup lang="ts">
import { ref } from 'vue';
import { useRouter } from 'vue-router';

import { useAuthStore } from '@/stores/auth.store';

const router = useRouter();
const authStore = useAuthStore();

const correo = ref('');
const password = ref('');
const error = ref('');
const cargando = ref(false);

async function iniciarSesion() {
  try {
    error.value = '';
    cargando.value = true;

    await authStore.login({
      correo: correo.value,
      password: password.value
    });

    router.push('/app');
  } catch {
    error.value = 'Credenciales incorrectas';
  } finally {
    cargando.value = false;
  }
}
</script>

<template>
  <main>
    <h1>Iniciar sesión</h1>

    <input
      v-model="correo"
      type="email"
      placeholder="Correo"
    />

    <input
      v-model="password"
      type="password"
      placeholder="Contraseña"
    />

    <button
      :disabled="cargando"
      @click="iniciarSesion"
    >
      {{ cargando ? 'Ingresando...' : 'Ingresar' }}
    </button>

    <p v-if="error">
      {{ error }}
    </p>
  </main>
</template>
```

En el App.vue:
```
<script setup lang="ts"></script>

<template>
  <RouterView />
</template>

<style scoped></style>
```


En router, abrir el archivo index.ts
```
import { createRouter, createWebHistory } from 'vue-router';

  

import LoginView from '@/views/LoginView.vue';

import DashboardView from '@/views/DashboardView.vue';

  

const router = createRouter({

history: createWebHistory(import.meta.env.BASE_URL),

  

routes: [

{

path: '/',

redirect: '/login'

},

{

path: '/login',

name: 'login',

component: LoginView

},

{

path: '/app',

name: 'app',

component: DashboardView

}

]

});

  

export default router;
```

En views, crear DashboardView.vue
```
<script setup lang="ts">
import { useAuthStore } from '@/stores/auth.store';

const authStore = useAuthStore();
</script>

<template>
  <main>
    <h1>Dashboard</h1>

    <p v-if="authStore.usuario">
      Bienvenido {{ authStore.usuario.nombre }}
    </p>
  </main>
</template>
```

El archivo App.vue, debe quedar así:
```
<script setup lang="ts"></script>

<template>
  <RouterView />
</template>

<style scoped></style>

```

EJECUTAR!!! pnpm run dev

Ahora, vamos a proteger la ruta app, para que nadie que no esté logueado pueda acceder
En el archivo router > index.ts agregar:
```import { useAuthStore } from '@/stores/auth.store'; ```

Debería quedar así:
```
import { createRouter, createWebHistory } from 'vue-router';

import { useAuthStore } from '@/stores/auth.store';

import LoginView from '@/views/LoginView.vue';
import DashboardView from '@/views/DashboardView.vue';

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),

  routes: [
    {
      path: '/',
      redirect: '/login'
    },
    {
      path: '/login',
      name: 'login',
      component: LoginView
    },
    {
      path: '/app',
      name: 'app',
      component: DashboardView,
      meta: {
        requiresAuth: true
      }
    }
  ]
});

router.beforeEach((to) => {
  const authStore = useAuthStore();

  const requiereAuth = to.meta.requiresAuth;

  if (requiereAuth && !authStore.estaAutenticado) {
    return '/login';
  }
});

export default router;
```

Ahora, en components, se crea el archivo AppNavbar.vue
```
<script setup lang="ts">
import { useRouter } from 'vue-router';
import { useAuthStore } from '@/stores/auth.store';

const router = useRouter();
const authStore = useAuthStore();

function cerrarSesion() {
  authStore.logout();
  router.push('/login');
}
</script>

<template>
  <nav>
    <strong>Mi Sistema</strong>

    <span v-if="authStore.usuario">
      {{ authStore.usuario.nombre }}
    </span>

    <button @click="cerrarSesion">
      Cerrar sesión
    </button>
  </nav>
</template>
```

Y en views > DashboardView.vue:
```
<script setup lang="ts">

import AppNavbar from '@/components/AppNavbar.vue';

import { useAuthStore } from '@/stores/auth.store';

  

const authStore = useAuthStore();

</script>

  

<template>

<AppNavbar />

<main>

<h1>Dashboard</h1>

  

<p v-if="authStore.usuario">

Bienvenido {{ authStore.usuario.nombre }}

</p>

</main>

</template>
```

Crear el package config, y dentro al archivo api.ts
```export const API_URL = 'http://localhost:8000';```

Ir a api > auth.api.ts y cambiar la primera línea por:
```import { API_URL } from '@/config/api';```

# Producto
En interfaces, crear producto.interface.ts
```
export interface Producto {
  id: number;
  nombre: string;
  descripcion: string | null;
  precio: number;
  stock: number;
  usuario_id: number;
}

export interface ProductoCrear {
  nombre: string;
  descripcion: string | null;
  precio: number;
  stock: number;
}

export interface ProductoActualizar {
  nombre: string;
  descripcion: string | null;
  precio: number;
  stock: number;
}
```

En api > producto.api.ts
```
import { API_URL } from '@/config/api';

import type {
  Producto,
  ProductoCrear,
  ProductoActualizar
} from '@/interfaces/producto.interface';

function getHeaders() {
  const token = localStorage.getItem('token');

  return {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`
  };
}

export async function listarProductos(): Promise<Producto[]> {

  const response = await fetch(
    `${API_URL}/productos/`,
    {
      headers: getHeaders()
    }
  );

  if (!response.ok) {
    throw new Error('Error al obtener productos');
  }

  return await response.json();
}

export async function crearProducto(
  producto: ProductoCrear
): Promise<Producto> {

  const response = await fetch(
    `${API_URL}/productos/`,
    {
      method: 'POST',
      headers: getHeaders(),
      body: JSON.stringify(producto)
    }
  );

  if (!response.ok) {
    throw new Error('Error al crear producto');
  }

  return await response.json();
}

export async function actualizarProducto(
  id: number,
  producto: ProductoActualizar
): Promise<Producto> {

  const response = await fetch(
    `${API_URL}/productos/${id}`,
    {
      method: 'PUT',
      headers: getHeaders(),
      body: JSON.stringify(producto)
    }
  );

  if (!response.ok) {
    throw new Error('Error al actualizar producto');
  }

  return await response.json();
}

export async function eliminarProducto(
  id: number
): Promise<void> {

  const response = await fetch(
    `${API_URL}/productos/${id}`,
    {
      method: 'DELETE',
      headers: getHeaders()
    }
  );

  if (!response.ok) {
    throw new Error('Error al eliminar producto');
  }
}

```

En views, crear ProductoView.vue
```
<script setup lang="ts">
import { onMounted, ref } from 'vue';

import AppNavbar from '@/components/AppNavbar.vue';
import { listarProductos } from '@/api/producto.api';

import type { Producto } from '@/interfaces/producto.interface';

const productos = ref<Producto[]>([]);
const error = ref('');
const cargando = ref(false);

async function cargarProductos() {
  try {
    cargando.value = true;
    error.value = '';

    productos.value = await listarProductos();
  } catch {
    error.value = 'Error al cargar productos';
  } finally {
    cargando.value = false;
  }
}

onMounted(() => {
  cargarProductos();
});
</script>

<template>
  <AppNavbar />

  <main>
    <h1>Mantenedor de Productos</h1>

    <p v-if="cargando">Cargando productos...</p>

    <p v-if="error">
      {{ error }}
    </p>

    <table v-if="productos.length > 0">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre</th>
          <th>Descripción</th>
          <th>Precio</th>
          <th>Stock</th>
        </tr>
      </thead>

      <tbody>
        <tr
          v-for="producto in productos"
          :key="producto.id"
        >
          <td>{{ producto.id }}</td>
          <td>{{ producto.nombre }}</td>
          <td>{{ producto.descripcion }}</td>
          <td>{{ producto.precio }}</td>
          <td>{{ producto.stock }}</td>
        </tr>
      </tbody>
    </table>

    <p v-else-if="!cargando">
      No hay productos registrados.
    </p>
  </main>
</template>
```

En Router > index.ts agregar
```import ProductoView from '@/views/ProductoView.vue';```

```
import ProductoView from '@/views/ProductoView.vue';


{
  path: '/app/productos',
  name: 'productos',
  component: ProductoView,
  meta: {
    requiresAuth: true
  }
}
```

En DashboardView.vue, dentro del main, agregar:
```
<RouterLink to="/app/productos">
  Ir al mantenedor de productos
</RouterLink>
```

Para crear producto
Dentro de ProductoView.vue, dentro de  script setup
```
const nombre = ref('');
const descripcion = ref('');
const precio = ref(0);
const stock = ref(0);
```

```
import {
  listarProductos,
  crearProducto
} from '@/api/producto.api';
```

```
async function guardarProducto() {
  try {
    error.value = '';

    await crearProducto({
      nombre: nombre.value,
      descripcion: descripcion.value,
      precio: precio.value,
      stock: stock.value
    });

    nombre.value = '';
    descripcion.value = '';
    precio.value = 0;
    stock.value = 0;

    await cargarProductos();
  } catch {
    error.value = 'Error al guardar producto';
  }
}
```

Ahora dentro del template, debajo del H1 agregar esto:
```
<section>
  <h2>Nuevo producto</h2>

  <input v-model="nombre" placeholder="Nombre" />

  <input v-model="descripcion" placeholder="Descripción" />

  <input
    v-model.number="precio"
    type="number"
    placeholder="Precio"
  />

  <input
    v-model.number="stock"
    type="number"
    placeholder="Stock"
  />

  <button @click="guardarProducto">
    Guardar
  </button>
</section>

<hr />
```

### Eliminar Producto
Agregar en el import de ProductoView.vue
```
import {  
listarProductos,  
crearProducto,  
eliminarProducto  
} from '@/api/producto.api';
```

Agregar la función
```
async function borrarProducto(id: number) {
  try {
    error.value = '';

    const confirmar = confirm('¿Seguro que deseas eliminar este producto?');

    if (!confirmar) {
      return;
    }

    await eliminarProducto(id);

    await cargarProductos();
  } catch {
    error.value = 'Error al eliminar producto';
  }
}
```

Modificar el head de la tabla:
```
<tr>
  <th>ID</th>
  <th>Nombre</th>
  <th>Descripción</th>
  <th>Precio</th>
  <th>Stock</th>
  <th>Acciones</th>
</tr>
```

Agregar botón eliminar
```
<td>
  <button @click="borrarProducto(producto.id)">
    Eliminar
  </button>
</td>
```

### Modificar Producto
En ProductoView.vue agregar en el import
```actualizarProducto ```

Agregar la siguiente variable:
```const productoEditandoId = ref<number | null>(null); ```

Agregar la siguiente función:
```
function editarProducto(producto: Producto) {
  productoEditandoId.value = producto.id;

  nombre.value = producto.nombre;
  descripcion.value = producto.descripcion || '';
  precio.value = producto.precio;
  stock.value = producto.stock;
}
```

Modificar la función guardarProducto
```
async function guardarProducto() {
  try {
    error.value = '';

    const datos = {
      nombre: nombre.value,
      descripcion: descripcion.value,
      precio: precio.value,
      stock: stock.value
    };

    if (productoEditandoId.value) {
      await actualizarProducto(productoEditandoId.value, datos);
    } else {
      await crearProducto(datos);
    }

    productoEditandoId.value = null;
    nombre.value = '';
    descripcion.value = '';
    precio.value = 0;
    stock.value = 0;

    await cargarProductos();
  } catch {
    error.value = 'Error al guardar producto';
  }
}
```

Cambiar el título:
```
<h2>  
{{ productoEditandoId ? 'Editar producto' : 'Nuevo producto' }}  
</h2>
```

Y el botón:
```
<button @click="guardarProducto">
  {{ productoEditandoId ? 'Actualizar' : 'Guardar' }}
</button>
```

Agregar botón en la tabla:
```
<button @click="editarProducto(producto)">  
Editar  
</button>
```

### Private Layout
Se crea la carpeta layouts
Se crea el archivo PrivateLayout.vue
```
<script setup lang="ts">
import AppNavbar from '@/components/AppNavbar.vue';
</script>

<template>
  <AppNavbar />

  <main>
    <RouterView />
  </main>
</template>
```

Dejar el DashboardView.vue así:
```
<script setup lang="ts">
import { useAuthStore } from '@/stores/auth.store';

const authStore = useAuthStore();
</script>

<template>
  <section>
    <h1>Dashboard</h1>

    <p v-if="authStore.usuario">
      Bienvenido {{ authStore.usuario.nombre }}
    </p>

    <RouterLink to="/app/productos">
      Ir al mantenedor de productos
    </RouterLink>
  </section>
</template>
```

En ProductoView.vue, quitar:
import AppNavbar from '@/components/AppNavbar.vue';
y esto
<AppNavbar />

El archivo router > index.ts debería quedar así:
```
import { createRouter, createWebHistory } from 'vue-router';

import { useAuthStore } from '@/stores/auth.store';

import LoginView from '@/views/LoginView.vue';
import DashboardView from '@/views/DashboardView.vue';
import ProductoView from '@/views/ProductoView.vue';

import PrivateLayout from '@/layouts/PrivateLayout.vue';

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),

  routes: [
    {
      path: '/',
      redirect: '/login'
    },
    {
      path: '/login',
      name: 'login',
      component: LoginView
    },
    {
      path: '/app',
      component: PrivateLayout,
      meta: {
        requiresAuth: true
      },
      children: [
        {
          path: '',
          name: 'app',
          component: DashboardView
        },
        {
          path: 'productos',
          name: 'productos',
          component: ProductoView
        }
      ]
    }
  ]
});

router.beforeEach((to) => {
  const authStore = useAuthStore();

  const requiereAuth = to.matched.some(
    (route) => route.meta.requiresAuth
  );

  if (requiereAuth && !authStore.estaAutenticado) {
    return '/login';
  }
});

export default router;

```

### CSS
Crear assets > main.css
```
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: #f4f6f8;
  color: #1f2937;
}

main {
  padding: 24px;
}

input {
  display: block;
  width: 100%;
  max-width: 400px;
  padding: 10px;
  margin-bottom: 12px;
}

button {
  padding: 10px 14px;
  cursor: pointer;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  margin-top: 16px;
}

th,
td {
  padding: 12px;
  border-bottom: 1px solid #ddd;
  text-align: left;
}

nav {
  height: 56px;
  padding: 0 24px;
  background: #111827;
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

nav a {
  color: white;
  margin-right: 16px;
}
```

El AppNavbar.vue queda:
```
<script setup lang="ts">

import { useRouter } from 'vue-router';

import { useAuthStore } from '@/stores/auth.store';

  

const router = useRouter();

const authStore = useAuthStore();

  

function cerrarSesion() {

authStore.logout();

router.push('/login');

}

</script>

  

<template>

<nav>

<div>

<strong>Mi Sistema</strong>

|

<RouterLink to="/app">Dashboard</RouterLink>

<RouterLink to="/app/productos">Productos</RouterLink>

</div>

  

<div>

<span v-if="authStore.usuario">

{{ authStore.usuario.nombre }}

</span>

  

<button @click="cerrarSesion">

Cerrar sesión

</button>

</div>

</nav>

</template>

```

y en el archivo main.ts agregar:
```import './assets/main.css' ```

### Tailwind
Instalar
```npm install tailwindcss @tailwindcss/vite ```

Abrir el archivo vite.config.ts
```
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueDevTools from 'vite-plugin-vue-devtools'
import tailwindcss from '@tailwindcss/vite'

// https://vite.dev/config/
export default defineConfig({
plugins: [
	vue(),
	vueDevTools(),
	tailwindcss()
],
	resolve: {
		alias: {
		'@': fileURLToPath(new URL('./src', import.meta.url))
		},
	},
})
```

Agregar Tailwinds al CSS principal
Abre src>assets>main.css y reemplaza todo por  ```@import "tailwindcss"; ```

El main.ts debería estar así:
```
import { createApp } from 'vue'
import { createPinia } from 'pinia'

import App from './App.vue'
import router from './router'
import './assets/main.css'

const app = createApp(App)

app.use(createPinia())
app.use(router)

app.mount('#app')
```

Reiniciar la aplicación para que tome la nueva importación de Tailwind

En DashboardView.vue, para probar, el h1 lo vamos a dejar así:
```
<h1 class="text-4xl font-bold text-blue-600">
  Dashboard
</h1>
```

Se debería ver el título con un nuevo estilo. Si es así, Tailwind quedó funcionando correctamente :)

---

Ahora se aplican los estilos al resto de archivos

PrivateLayout.vue
```
<script setup lang="ts">
import AppNavbar from '@/components/AppNavbar.vue';
</script>

<template>
  <div class="min-h-screen bg-slate-100">
    <AppNavbar />

    <main class="mx-auto max-w-7xl px-6 py-8">
      <RouterView />
    </main>
  </div>
</template>
```

AppNavbar.vue
```
<script setup lang="ts">
import { useRouter } from 'vue-router';
import { useAuthStore } from '@/stores/auth.store';

const router = useRouter();
const authStore = useAuthStore();

function cerrarSesion() {
  authStore.logout();
  router.push('/login');
}
</script>

<template>
  <nav class="bg-slate-900 text-white shadow">
    <div class="mx-auto flex h-16 max-w-7xl items-center justify-between px-6">
      <div class="flex items-center gap-6">
        <strong class="text-lg">Mi Sistema</strong>

        <RouterLink class="text-slate-300 hover:text-white" to="/app">
          Dashboard
        </RouterLink>

        <RouterLink class="text-slate-300 hover:text-white" to="/app/productos">
          Productos
        </RouterLink>
      </div>

      <div class="flex items-center gap-4">
        <span v-if="authStore.usuario" class="text-sm text-slate-300">
          {{ authStore.usuario.nombre }}
        </span>

        <button
          class="rounded bg-red-600 px-4 py-2 text-sm font-medium hover:bg-red-700"
          @click="cerrarSesion"
        >
          Cerrar sesión
        </button>
      </div>
    </div>
  </nav>
</template>
```

LoginView.vue
Reemplazar el template por esto
```
<template>
  <main class="flex min-h-screen items-center justify-center bg-slate-100 px-4">
    <section class="w-full max-w-md rounded-xl bg-white p-8 shadow">
      <h1 class="mb-6 text-center text-3xl font-bold text-slate-800">
        Iniciar sesión
      </h1>

      <div class="space-y-4">
        <input
          v-model="correo"
          class="w-full rounded border border-slate-300 px-4 py-2 outline-none focus:border-blue-500"
          type="email"
          placeholder="Correo"
        />

        <input
          v-model="password"
          class="w-full rounded border border-slate-300 px-4 py-2 outline-none focus:border-blue-500"
          type="password"
          placeholder="Contraseña"
        />

        <button
          class="w-full rounded bg-blue-600 px-4 py-2 font-medium text-white hover:bg-blue-700 disabled:bg-blue-300"
          :disabled="cargando"
          @click="iniciarSesion"
        >
          {{ cargando ? 'Ingresando...' : 'Ingresar' }}
        </button>

        <p v-if="error" class="text-center text-sm text-red-600">
          {{ error }}
        </p>
      </div>
    </section>
  </main>
</template>
```

DashboardView.vue
```
<script setup lang="ts">
import { useAuthStore } from '@/stores/auth.store';

const authStore = useAuthStore();
</script>

<template>
  <section>
    <div class="rounded-xl bg-white p-8 shadow">
      <h1 class="text-3xl font-bold text-slate-800">
        Dashboard
      </h1>

      <p v-if="authStore.usuario" class="mt-2 text-slate-600">
        Bienvenido {{ authStore.usuario.nombre }}
      </p>

      <RouterLink
        class="mt-6 inline-block rounded bg-blue-600 px-4 py-2 font-medium text-white hover:bg-blue-700"
        to="/app/productos"
      >
        Ir al mantenedor de productos
      </RouterLink>
    </div>
  </section>
</template>
```

ProductoView.vue
Reemplazar el template por
```
<template>
  <section class="space-y-6">
    <div>
      <h1 class="text-3xl font-bold text-slate-800">
        Mantenedor de Productos
      </h1>
      <p class="mt-1 text-slate-500">
        Administra tus productos registrados.
      </p>
    </div>

    <section class="rounded-xl bg-white p-6 shadow">
      <h2 class="mb-4 text-xl font-semibold text-slate-800">
        {{ productoEditandoId ? 'Editar producto' : 'Nuevo producto' }}
      </h2>

      <div class="grid gap-4 md:grid-cols-2">
        <input
          v-model="nombre"
          class="rounded border border-slate-300 px-4 py-2 outline-none focus:border-blue-500"
          placeholder="Nombre"
        />

        <input
          v-model="descripcion"
          class="rounded border border-slate-300 px-4 py-2 outline-none focus:border-blue-500"
          placeholder="Descripción"
        />

        <input
          v-model.number="precio"
          class="rounded border border-slate-300 px-4 py-2 outline-none focus:border-blue-500"
          type="number"
          placeholder="Precio"
        />

        <input
          v-model.number="stock"
          class="rounded border border-slate-300 px-4 py-2 outline-none focus:border-blue-500"
          type="number"
          placeholder="Stock"
        />
      </div>

      <button
        class="mt-4 rounded bg-blue-600 px-4 py-2 font-medium text-white hover:bg-blue-700"
        @click="guardarProducto"
      >
        {{ productoEditandoId ? 'Actualizar' : 'Guardar' }}
      </button>
    </section>

    <p v-if="cargando" class="text-slate-600">
      Cargando productos...
    </p>

    <p v-if="error" class="rounded bg-red-100 px-4 py-3 text-red-700">
      {{ error }}
    </p>

    <section class="overflow-hidden rounded-xl bg-white shadow">
      <table v-if="productos.length > 0" class="w-full border-collapse">
        <thead class="bg-slate-800 text-white">
          <tr>
            <th class="px-4 py-3 text-left">ID</th>
            <th class="px-4 py-3 text-left">Nombre</th>
            <th class="px-4 py-3 text-left">Descripción</th>
            <th class="px-4 py-3 text-left">Precio</th>
            <th class="px-4 py-3 text-left">Stock</th>
            <th class="px-4 py-3 text-left">Acciones</th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="producto in productos"
            :key="producto.id"
            class="border-b last:border-b-0 hover:bg-slate-50"
          >
            <td class="px-4 py-3">{{ producto.id }}</td>
            <td class="px-4 py-3 font-medium text-slate-800">
              {{ producto.nombre }}
            </td>
            <td class="px-4 py-3 text-slate-600">
              {{ producto.descripcion || 'Sin descripción' }}
            </td>
            <td class="px-4 py-3">
              ${{ producto.precio }}
            </td>
            <td class="px-4 py-3">
              {{ producto.stock }}
            </td>
            <td class="space-x-2 px-4 py-3">
              <button
                class="rounded bg-amber-500 px-3 py-1 text-sm font-medium text-white hover:bg-amber-600"
                @click="editarProducto(producto)"
              >
                Editar
              </button>

              <button
                class="rounded bg-red-600 px-3 py-1 text-sm font-medium text-white hover:bg-red-700"
                @click="borrarProducto(producto.id)"
              >
                Eliminar
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <p v-else-if="!cargando" class="p-6 text-center text-slate-500">
        No hay productos registrados.
      </p>
    </section>
  </section>
</template>
```

