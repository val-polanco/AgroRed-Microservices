# AgroRed Microservices

Plataforma web de gestión comercial construida con una arquitectura de microservicios para administrar productos, clientes, pedidos y usuarios.

El proyecto integra backend, frontend, autenticación JWT, bases de datos MySQL, API Gateway con Apache, DNS con BIND9 y despliegue de servicios Node.js mediante systemd en Linux.

> Proyecto académico desarrollado originalmente de forma colaborativa para la asignatura Redes y Servicios Multimedia. Esta versión está organizada como proyecto de portafolio y documenta especialmente mi participación en el desarrollo de `ms-pedidos`, integración entre servicios, seguridad, despliegue e integración final.

---

## Descripción

AgroRed simula una plataforma de gestión comercial para una empresa dedicada a la distribución de productos agrícolas.

La aplicación permite administrar:

- Productos.
- Clientes.
- Pedidos.
- Usuarios.
- Roles y permisos.
- Relaciones entre pedidos, clientes y productos.

El sistema fue diseñado utilizando una arquitectura distribuida donde cada recurso principal tiene su propio microservicio y base de datos.

---

## Arquitectura

La aplicación utiliza dos servidores Linux virtualizados.

### Servidor 1

Responsable de infraestructura y datos:

- MySQL.
- BIND9.
- DNS interno.
- Bases de datos independientes por microservicio.

Bases de datos:

- `usuarios_db`
- `productos_db`
- `clientes_db`
- `pedidos_db`

### Servidor 2

Responsable de la aplicación:

- Node.js.
- Microservicios.
- Apache.
- API Gateway.
- Frontend.
- Servicios systemd.

Microservicios:

- `ms-usuarios` → puerto `3001`
- `ms-productos` → puerto `3002`
- `ms-clientes` → puerto `3003`
- `ms-pedidos` → puerto `3004`

Los microservicios escuchan únicamente en `127.0.0.1` y no están expuestos directamente al cliente.

---

## Flujo general

```text
Usuario
   │
   ▼
app.agrored.com
   │
   ▼
BIND9 DNS
   │
   ▼
Apache :80
   │
   ▼
API Gateway
   │
   ├── /api/usuarios  → ms-usuarios
   ├── /api/productos → ms-productos
   ├── /api/clientes  → ms-clientes
   └── /api/pedidos   → ms-pedidos
                            │
                            ├── ms-clientes
                            └── ms-productos
   │
   ▼
MySQL
```

---

## Tecnologías

### Backend

- Node.js
- Express.js
- JavaScript
- REST API
- JWT
- bcrypt
- MySQL

### Frontend

- HTML5
- CSS3
- JavaScript
- Fetch API
- Session Storage

### Infraestructura

- Ubuntu Server
- Apache HTTP Server
- Apache `mod_proxy`
- BIND9
- systemd
- UFW
- Vagrant

### Control de versiones

- Git
- GitHub

---

## API Gateway

El cliente no necesita conocer los puertos internos de cada microservicio.

Apache funciona como proxy inverso y expone una única API:

```text
http://app.agrored.com/api
```

Ejemplos:

```text
/api/usuarios
/api/productos
/api/clientes
/api/pedidos
```

Internamente Apache redirige cada solicitud al microservicio correspondiente.

---

## Autenticación y seguridad

La aplicación utiliza autenticación mediante JSON Web Tokens.

El flujo es:

```text
Login
  ↓
ms-usuarios
  ↓
Validación de credenciales
  ↓
JWT
  ↓
sessionStorage
  ↓
Authorization: Bearer <token>
  ↓
Microservicios protegidos
```

Los microservicios verifican el token antes de permitir acceso a las rutas protegidas.

También existen permisos basados en roles.

---

## Microservicio de pedidos

`ms-pedidos` administra las operaciones relacionadas con pedidos.

Entre sus responsabilidades se encuentran:

- Crear pedidos.
- Consultar pedidos.
- Actualizar pedidos.
- Eliminar pedidos.
- Validar referencias de clientes.
- Validar referencias de productos.
- Verificar autenticación JWT.
- Aplicar permisos por rol.
- Componer respuestas con información relacionada.

Un pedido puede retornar información como:

```json
{
  "id": 1,
  "cliente_id": 1,
  "producto_id": 1,
  "cantidad": 120,
  "centro_distribucion": "Cali",
  "estado": "despachado",
  "cliente": {
    "id": 1,
    "nombre": "Distribuidora El Valle"
  },
  "producto": {
    "id": 1,
    "nombre": "Mango Tommy"
  }
}
```

Esto permite que el frontend reciba una respuesta compuesta sin tener que consultar varios servicios por separado.

---

## Comunicación entre microservicios

`ms-pedidos` se comunica internamente con:

```text
ms-clientes
ms-productos
```

Las llamadas se realizan dentro de servidor2 utilizando direcciones locales.

Ejemplo:

```text
ms-pedidos
    │
    ├── http://127.0.0.1:3003 → ms-clientes
    │
    └── http://127.0.0.1:3002 → ms-productos
```

---

## Despliegue con systemd

Cada microservicio funciona como un servicio del sistema.

Esto permite:

- Arranque automático.
- Reinicio automático si el proceso falla.
- Ejecución sin mantener una terminal abierta.
- Administración con `systemctl`.
- Consulta de logs mediante `journalctl`.

Ejemplo:

```bash
sudo systemctl status ms-pedidos
sudo systemctl restart ms-pedidos
journalctl -u ms-pedidos
```

---

## Estructura del proyecto

```text
AgroRed-Microservices/
│
├── deploy/
│   ├── agrored-gateway.conf
│   ├── app.conf
│   ├── bases-de-datos.sql
│   ├── db.agrored.com
│   ├── ms-clientes.service
│   ├── ms-pedidos.service
│   ├── ms-productos.service
│   └── ms-usuarios.service
│
├── frontend/
│   ├── assets/
│   ├── data/
│   ├── js/
│   ├── clientes.html
│   ├── index.html
│   ├── inicio.html
│   ├── pedidos.html
│   ├── productos.html
│   └── usuarios.html
│
├── ms-clientes/
├── ms-pedidos/
├── ms-productos/
├── ms-usuarios/
│
└── .gitignore
```

---

## Variables de entorno

Cada microservicio utiliza su propio archivo `.env`.

Los archivos `.env` reales no forman parte del repositorio.

Cada servicio incluye un `.env.example` con la estructura requerida.

Ejemplo:

```env
PORT=3004
HOST=127.0.0.1

DB_HOST=IP_SERVIDOR_MYSQL
DB_PORT=3306
DB_NAME=pedidos_db
DB_USER=pedidos_user
DB_PASSWORD=CAMBIAR_EN_PRODUCCION

JWT_SECRET=CAMBIAR_EN_PRODUCCION

CLIENTES_URL=http://127.0.0.1:3003
PRODUCTOS_URL=http://127.0.0.1:3002
```

---

## Frontend

El frontend consume exclusivamente el API Gateway utilizando rutas relativas.

Ejemplo:

```javascript
fetch('/api/pedidos')
```

Las solicitudes autenticadas incluyen:

```text
Authorization: Bearer <token>
```

La aplicación incluye pantallas para:

- Inicio de sesión.
- Dashboard.
- Pedidos.
- Clientes.
- Productos.
- Usuarios.

---

## Funcionalidades principales

- Inicio de sesión real.
- Autenticación JWT.
- Control de acceso por roles.
- CRUD de productos.
- CRUD de clientes.
- CRUD de pedidos.
- CRUD de usuarios.
- API Gateway.
- Comunicación entre microservicios.
- Datos compuestos.
- DNS interno.
- Bases de datos independientes.
- Servicios systemd.
- Frontend conectado a la API real.

---

## Mi contribución

Mi participación se concentró principalmente en:

- Desarrollo de `ms-pedidos`.
- Implementación de seguridad JWT.
- Configuración de permisos por rol.
- Integración de pedidos con clientes.
- Integración de pedidos con productos.
- Validación de referencias entre microservicios.
- Composición de respuestas con datos relacionados.
- Configuración de MySQL para el entorno distribuido.
- Despliegue de microservicios en Linux.
- Configuración y pruebas con systemd.
- Integración con Apache API Gateway.
- Pruebas del frontend contra la API real.
- Resolución de problemas de red entre servidores.
- Integración final de ramas Git.
- Consolidación del repositorio para despliegue.

---

## Aprendizajes

Este proyecto permitió aplicar conceptos como:

- Arquitectura de microservicios.
- Comunicación entre servicios.
- APIs REST.
- Seguridad mediante JWT.
- Separación de bases de datos.
- Reverse proxy.
- API Gateway.
- DNS.
- Administración de servicios Linux.
- Despliegue de aplicaciones Node.js.
- Resolución de problemas de red.
- Integración de código mediante Git.

---

## Autoría

Proyecto académico desarrollado originalmente en equipo.

Versión de portafolio preparada por:

**Valeria Polanco**

Ingeniería Multimedia

GitHub: `val-polanco`

---

## Estado

Proyecto funcional.

La aplicación fue desplegada y validada utilizando:

```text
Windows
   ↓
DNS
   ↓
Apache API Gateway
   ↓
Microservicios Node.js
   ↓
MySQL
```
