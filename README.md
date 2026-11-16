# Casino online con ruleta europea

Aplicación web full-stack que simula un casino online con ruleta europea. Proyecto académico desarrollado en equipo de dos integrantes durante agosto – noviembre de 2025, en la Universidad Diego Portales (ramo CIT2008).

> **Aviso:** es un proyecto educativo. Todo el dinero es ficticio: los depósitos y retiros son simulados y no se procesa ningún pago real.

## Capturas

### Cuenta

| Registro | Validación de mayoría de edad | Inicio de sesión |
|---|---|---|
| ![Registro](capturas/01-registro.jpeg) | ![Registro con error de edad](capturas/01b-registro-error.jpeg) | ![Inicio de sesión](capturas/02-login.jpeg) |

### Perfil y transacciones

| Perfil con movimientos | Depósito exitoso |
|---|---|
| ![Perfil](capturas/03-perfil.jpeg) | ![Depósito exitoso](capturas/04-transacciones.jpeg) |

### Ruleta

| Tablero con apuestas | Resultado de un giro |
|---|---|
| ![Tablero de la ruleta](capturas/05-ruleta-tablero.jpeg) | ![Resultado de un giro](capturas/06-ruleta-resultado.jpeg) |


## Funcionalidades

- Registro de usuarios con validación de mayoría de edad.
- Inicio y cierre de sesión con contraseñas cifradas (bcrypt) y autenticación con JWT en una cookie `httpOnly`.
- Perfil con saldo, depósitos y retiros, e historial de movimientos.
- Ruleta europea con distintos tipos de apuesta (número, docena, columna, color, par/impar, mitades, entre otras). El número ganador se sortea y los pagos se calculan en el servidor.
- Tablero interactivo con apuestas sobre números y entre casillas, fichas de distintos valores, y bloqueo de nuevas apuestas mientras la ruleta gira.
- Historial de apuestas por usuario.
- El saldo se valida siempre en el servidor, por lo que se mantiene consistente aunque haya varias pestañas abiertas: un retiro impide girar la ruleta si el saldo ya no alcanza, y un giro que deja el saldo en cero impide retirar.

## Tecnologías

- **Frontend:** HTML, CSS y JavaScript.
- **Backend:** Node.js y Express (API REST).
- **Base de datos:** MongoDB, con Mongoose.
- **Seguridad:** bcrypt para contraseñas, JWT con expiración de 10 minutos, cookie `httpOnly`, `secure` y `sameSite: strict`.
- **Despliegue:** servidor Linux en AWS EC2, con Nginx y HTTPS.

## Estructura del repositorio

```
.
├── backend/
│   ├── server.js          # Servidor Express: configuración, rutas y archivos del frontend
│   ├── routes/
│   │   ├── index.js       # Agrupa las rutas bajo /api
│   │   ├── auth.js        # Registro, inicio y cierre de sesión
│   │   ├── user.js        # Perfil, depósitos y retiros
│   │   └── game.js        # Giro de la ruleta e historial de apuestas
│   └── Utils/
│       ├── db.js          # Conexión a MongoDB y modelo de usuario
│       ├── jwt.js         # Generación y verificación de tokens
│       └── payments.js    # Cuotas de cada apuesta y cálculo de pagos
├── frontend/              # Páginas, estilos, scripts e imágenes
├── capturas/              # Capturas de la aplicación en funcionamiento
├── .env.example           # Variables de entorno necesarias (con valores de ejemplo)
└── README.md
```

## API

Todas las rutas cuelgan de `/api`. Las que requieren sesión usan la cookie con el JWT.

| Método | Ruta | Descripción | Sesión |
|---|---|---|---|
| POST | `/auth/register` | Crea un usuario | No |
| POST | `/auth/login` | Inicia sesión | No |
| POST | `/auth/logout` | Cierra sesión | No |
| GET | `/user/profile` | Datos del usuario, saldo y movimientos | Sí |
| POST | `/user/deposit` | Registra un depósito | Sí |
| POST | `/user/withdraw` | Registra un retiro | Sí |
| POST | `/game/spin` | Juega una ronda de ruleta y calcula el pago | Sí |
| GET | `/game/history` | Historial de apuestas | Sí |

## Variables de entorno

La aplicación lee estas variables de entorno (hay un ejemplo en `.env.example`):

| Variable | Descripción |
|---|---|
| `MONGODB_URI` | Cadena de conexión a una base de datos MongoDB |
| `JWT_SECRET` | Texto largo y aleatorio para firmar los tokens |

## Sobre este repositorio

Este repositorio contiene únicamente el código desarrollado por nosotros, publicado como un único commit. El repositorio original de la sección es privado.

- `backend/server.js` parte de la base entregada por el profesor de la sección y fue extendido por nosotros para conectar la base de datos, las cookies, las rutas de la API y el servicio del frontend.
- La configuración de despliegue (Nginx, PM2, script de instalación) y los linters pertenecen al template creado por el profesor Pedro Toledo para su sección del ramo, y no se incluyen aquí.
- Las credenciales que había en el código se reemplazaron por variables de entorno.
- Por lo anterior, este repositorio **no se ejecuta por sí solo** (faltan, por ejemplo, el `package.json` y la configuración de despliegue). Las capturas de la carpeta `capturas/` muestran la aplicación funcionando.

## Equipo y aportes

| Integrante | Aporte |
|---|---|
| Eduardo Vergara Mora ([@EduardoVergaraM](https://github.com/EduardoVergaraM)) | **Backend:** modelado de datos en MongoDB (esquema de usuarios con sus movimientos y apuestas, con Mongoose, en `db.js`); autenticación con registro de usuarios, creación del JWT y manejo de la cookie de sesión (`auth.js`, `jwt.js`); perfil, transacciones, depósitos y retiros, con validación del saldo (`user.js`). **Frontend:** scripts de registro, inicio de sesión, perfil, transacciones y manejo de la sesión (`login.js`, `registro.js`, `perfil.js`, `transacciones.js`, `commons.js`). |
| Alexander Bravo ([@Aleesk](https://github.com/Aleesk)) | **La ruleta completa:** movimiento de la ruleta con CSS, tablero interactivo (apuestas sobre casillas y entre casillas), fichas de distintos valores con modales para ingresar montos y bloqueo de apuestas mientras gira (`ruleta.js`, `ruleta.css`); tipos de apuesta, giro y cálculo de pagos en el servidor (`game.js`, `payments.js`); estilos del sitio (`style.css`, en su totalidad); conexión de la aplicación al cluster de MongoDB. |

El HTML, la consistencia del saldo entre retiros y giros (que se valida en `user.js` y en `game.js`), el despliegue en AWS EC2 y la integración de las partes se hicieron en conjunto.

## Limitaciones y posibles mejoras

- La interfaz solo permite montos positivos, pero el servidor no lo vuelve a comprobar en el giro de la ruleta: una petición directa a la API podría enviar montos inválidos.
- El saldo se lee y se actualiza en pasos separados; con peticiones simultáneas convendría usar actualizaciones atómicas de MongoDB.
- La sesión expira a los 10 minutos y no hay renovación automática del token.
- Agregar pruebas automatizadas y un `package.json` propio para que el repositorio se pueda ejecutar de forma independiente.

## Créditos

- Proyecto realizado junto a Alexander Bravo en la Universidad Diego Portales.
- Base del servidor y configuración de despliegue: template creado por el profesor Pedro Toledo para su sección del ramo CIT2008.
