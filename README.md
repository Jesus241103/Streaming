# STREAMING — Sistema de Gestión de Cuentas

[![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-4.21-000000?logo=express&logoColor=white)](https://expressjs.com/)
[![MySQL](https://img.shields.io/badge/MySQL-5.7%2B-4479A1?logo=mysql&logoColor=white)](https://www.mysql.com/)
[![License](https://img.shields.io/badge/License-Propietaria-red.svg)](#-licencia)

> Panel web para administrar cuentas de streaming, clientes, vencimientos, renovaciones y respaldos. Pensado para operadores que necesitan llevar un control centralizado de stock de cuentas, asignaciones a clientes y reportes operativos.

---

## Tabla de contenidos

- [Descripción](#descripción)
- [Características](#características)
- [Arquitectura y stack](#arquitectura-y-stack)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Requisitos previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Roles y permisos](#roles-y-permisos)
- [Respaldos](#respaldos)
- [Licencia](#licencia)

---

## Descripción

**STREAMING** es una aplicación web full-stack que centraliza la gestión operativa de un negocio de reventa de cuentas de streaming. Permite registrar el inventario de cuentas disponibles, asignarlas a clientes con fechas de inicio y vencimiento, renovar suscripciones de forma individual o masiva, importar/exportar datos en hojas de cálculo y mantener un historial auditable de eliminaciones y renovaciones.

El sistema está construido sobre **Express 5** con autenticación basada en **JSON Web Tokens en cookies HTTP-only** y persistencia en **MySQL/MariaDB**. El frontend es una SPA ligera servida estáticamente, sin frameworks pesados, lo que mantiene el despliegue simple y los tiempos de carga bajos.

---

## Características

### Gestión de clientes
- Alta, edición y baja de clientes con validación de fechas.
- Asignación de **una o múltiples cuentas** a un mismo cliente desde el formulario.
- Atajos de fecha para extender vencimientos en +1, +2 o +3 meses.
- Búsqueda por lotes (batch lookup) por correo o ID de cliente.
- Estado activo/inactivo por registro.

### Operaciones masivas
- Carga masiva de cuentas en stock.
- Reemplazo masivo de datos de cuenta.
- Renovación masiva por tipo o selección.
- Eliminación masiva con registro automático de motivo.
- Importación/exportación de clientes en formato **Excel (.xlsx)**.

### Inventario de cuentas
- Catálogo de cuentas disponibles separado de los clientes.
- Categoría especial para cuentas con incidencias.

### Resumen y reportes
- Métricas agregadas (totales por tipo, próximos a vencer, etc.).
- Logs persistentes de **eliminaciones** y **renovaciones** consultables y descargables.

### Administración
- Gestión de usuarios con roles `admin` y `user`.
- Sistema de respaldos: creación, listado, restauración y eliminación de copias.
- Sesiones con expiración automática y limpieza de cookie al expirar.

### Experiencia de usuario
- Tema claro / oscuro con toggle persistente.
- Notificaciones con SweetAlert2.
- Iconografía con Font Awesome.
- Diseño responsive con sidebar fija.

---

## Arquitectura y stack

| Capa            | Tecnología                                |
| --------------- | ----------------------------------------- |
| Runtime         | Node.js (>= 18)                           |
| Framework HTTP  | Express 5                                 |
| Base de datos   | MySQL 8 / MariaDB 10.4+                   |
| Driver DB       | `mysql2` (pool de conexiones, promesas)   |
| Autenticación   | `jsonwebtoken` + `cookie-parser`          |
| Hashing         | `bcrypt`                                  |
| Subida archivos | `multer`                                  |
| Excel           | `xlsx`                                    |
| Variables env   | `dotenv`                                  |
| Frontend        | HTML5, CSS3, JavaScript vanilla           |
| UI helpers      | SweetAlert2, Font Awesome, Google Fonts   |

---

## Estructura del proyecto

```
NETFLIX/
├── backups/                # Copias generadas por el panel de respaldos
├── middleware/             # Middlewares de autenticación
├── public/                 # Frontend estático servido por Express
│   ├── login (vista pública)
│   ├── dashboard (vista privada post-login)
│   └── src/                # Recursos estáticos
├── connexion.js            # Pool de conexiones a base de datos
├── endpoints.js            # Definición de rutas y lógica de negocio
├── index.js                # Entry point del servidor
├── package.json
└── .env                    # Variables de entorno (NO versionar)
```

> Por seguridad, el dump SQL inicial, el archivo `.env` y el contenido de `backups/` no deben publicarse en repositorios remotos.

---

## Requisitos previos

- **Node.js** 18 o superior.
- **npm** 9 o superior (incluido con Node).
- **MySQL 8** o **MariaDB 10.4+** corriendo localmente o accesible por red.
- (Opcional) cliente gráfico tipo phpMyAdmin/Workbench para administrar la base.

---

## Instalación

1. **Obtener el código** (clonado o copia local):

   ```bash
   git clone <url-del-repositorio>
   cd NETFLIX
   ```

2. **Instalar dependencias**:

   ```bash
   npm install
   ```

3. **Crear la base de datos** a partir del script SQL provisto por el responsable del sistema. El script genera las tablas necesarias y un usuario administrador inicial.

4. **Configurar variables de entorno**: ver sección [Configuración](#configuración).

5. **Iniciar el servidor** en modo desarrollo (con auto-reload):

   ```bash
   npm run dev
   ```

   El servidor quedará disponible en el puerto configurado (por defecto `3000`).

---

## Configuración

Crear un archivo `.env` en la raíz del proyecto con las variables necesarias para:

- Conexión a la base de datos (host, usuario, contraseña, nombre, puerto y límite de conexiones).
- Secreto para la firma de tokens de sesión.
- Puerto del servidor (opcional).

> **Importante**
> - El secreto de sesión debe ser una cadena aleatoria larga y única en producción.
> - El archivo `.env` está incluido en `.gitignore` y **no debe subirse al repositorio**.
> - Las credenciales de acceso del usuario administrador inicial deben **cambiarse en el primer inicio de sesión** desde el panel de Usuarios.

Las variables exactas y los valores de ejemplo deben solicitarse al responsable del sistema o consultarse en la documentación interna.

---

## Uso

1. Abrir el sistema en el navegador e iniciar sesión con un usuario válido.
2. El dashboard expone las siguientes secciones en el sidebar:
   - **Resumen General** — métricas y totales.
   - **Clientes** — alta, edición, renovación, importación/exportación.
   - **Editor** — gestión del stock de cuentas.
   - **Usuarios** — alta de operadores y administradores (solo admin).
   - **Respaldos** — crear, restaurar y descargar respaldos (solo admin).
   - **Registros Eliminados** — auditoría de bajas (solo admin).
3. Las acciones masivas se invocan desde botones contextuales en cada vista.
4. El cierre de sesión limpia la cookie de acceso y redirige al login.

---

## Roles y permisos

- **`user`** — acceso al dashboard, consultas, renovaciones individuales/masivas y operaciones no destructivas.
- **`admin`** — todo lo anterior más: gestión de usuarios, respaldos, logs y operaciones masivas destructivas.

El acceso al sistema se valida mediante un middleware de autenticación que verifica el token de sesión en cookies. Las solicitudes no autenticadas son rechazadas o redirigidas al login según el contexto.

---

## Respaldos

Los archivos de respaldo se almacenan en la carpeta `backups/` del proyecto. **No deben versionarse** ni exponerse públicamente. En entornos productivos conviene rotarlos y sincronizarlos a almacenamiento externo (S3, rclone, NAS, etc.).

---

## Licencia

**Software propietario — Todos los derechos reservados.**

Copyright © 2026. Este software es propiedad exclusiva de su autor. Su código fuente, diseño, documentación y demás componentes están protegidos por las leyes de propiedad intelectual y derechos de autor aplicables.

Queda **estrictamente prohibido**, sin autorización previa, expresa y por escrito del propietario:

- Copiar, reproducir, distribuir o publicar total o parcialmente el código o sus componentes.
- Modificar, adaptar, traducir o crear obras derivadas a partir de este software.
- Realizar ingeniería inversa, descompilar o desensamblar el sistema.
- Sublicenciar, vender, alquilar, ceder o transferir el software o cualquier derecho sobre él.
- Utilizar el software con fines comerciales o no comerciales fuera del alcance autorizado por el propietario.

El acceso al código no implica la cesión de ningún derecho sobre el mismo. Cualquier uso no autorizado constituirá una violación de los derechos de propiedad intelectual y será perseguido conforme a la legislación vigente.

Para solicitar permisos, licencias de uso o consultas legales, contactar directamente con el propietario del proyecto.

---

## Contacto

Para consultas, soporte o solicitudes de licencia, contactar directamente con el propietario del proyecto.

---

<p align="center">
  martj2024@gmail.com
</p>