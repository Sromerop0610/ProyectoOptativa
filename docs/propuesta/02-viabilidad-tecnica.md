# VIABILIDAD TÉCNICA

## REQUISITOS TÉCNICOS
### FRONTEND (React)

El frontend será la parte de la aplicación con la que interactuarán los niños y sus padres o tutores. Se desarrollará con React y se añadirán bibliotecas que faciliten la navegación, la comunicación con el backend y el diseño de la interfaz.

Hemos pensado en usar:

| Biblioteca         | ¿Para qué la usaremos?                                                                                  | ¿Qué hace?                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **React Router**   | Gestionar las páginas de la aplicación: inicio de sesión, perfil infantil, panel de padres y registros. | Permite navegar entre páginas sin recargar toda la aplicación.                                         |
| **TanStack Query** | Consultar, guardar y actualizar datos obtenidos del backend, como los registros de glucosa.             | Facilita la gestión de las peticiones, los estados de carga y los errores.                             |
| **Tailwind CSS**   | Diseñar botones, formularios, paneles y tarjetas adaptables a distintos tamaños de pantalla.            | Permite crear una interfaz visual y personalizable sin escribir demasiado CSS repetitivo.              |
| **Recharts**       | Representar gráficamente el historial de glucosa y su evolución.                                        | Facilita la creación de gráficos para que los padres puedan interpretar los registros de forma visual. |

### INFRAESTRUCTURA

Proponemos separar el frontend, el backend y la base de datos. De esta manera, cada parte podrá desplegarse y actualizarse de forma independiente.

#### Servicios elegidos y sus condiciones gratuitas

| **Servicio**           | **Uso**                          | **Condiciones del plan gratuito**                                                                                                                                                                                                       |
| ---------------------- | -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Vercel Hobby**       | Desplegar el frontend de React.  | Incluye alojamiento para proyectos personales, con límites de uso. Entre sus cuotas publicadas están 1 millón de invocaciones de funciones y 100.000 lecturas de Edge Config por periodo de facturación, si se utilizan esas funciones. |
| **Render Free**        | Ejecutar la API de Express.      | Dispone de 750 horas de instancia gratuitas por espacio de trabajo y mes. El servicio se suspende tras 15 minutos sin tráfico entrante y tarda aproximadamente un minuto en volver a arrancar al recibir una petición.                  |
| **MongoDB Atlas Free** | Alojar la base de datos MongoDB. | Ofrece 512 MB de almacenamiento. El clúster gratuito tiene límites de rendimiento y no incluye las copias de seguridad gestionadas disponibles en planes superiores.                                                                    |

Los planes y sus condiciones pueden cambiar, así que conviene volver a comprobarlos antes de desplegar el proyecto.

No necesitamos contratar un servidor propio para la primera versión. Vercel, Render y MongoDB Atlas cubren el alojamiento de la interfaz, la ejecución del backend y el almacenamiento de los datos.

Sí necesitaremos configurar algunas medidas adicionales:

* Variables de entorno para guardar la cadena de conexión de MongoDB y las claves secretas del backend.
* Conexiones HTTPS para proteger la comunicación.
* Restricciones de acceso a MongoDB para que solo los servicios autorizados puedan conectarse.
* Validación de las peticiones y comprobación de permisos en el backend.
* Una política de copias de seguridad y recuperación antes de considerar el uso de datos reales.

El principal inconveniente de Render Free es que el backend puede tardar en responder cuando vuelve a arrancar tras un periodo de inactividad. Para una demostración en clase es asumible, pero debe tenerse en cuenta.

## Requisitos técnicos: backend y base de datos
### Backend(Node.js + express)
#### 1. Autenticación y Roles

La aplicación implementa un sistema de autenticación con control de acceso basado en roles (**RBAC**):

* **Niño:** Puede registrar sus datos diarios (glucosa, comidas), consultar su historial y acceder al contenido didáctico.
* **Adulto (Padre / Tutor):** Puede crear o vincular perfiles infantiles, consultar los registros del menor y configurar los parámetros médicos de la calculadora.

El backend verifica la autenticación, el rol del usuario y su relación con el perfil infantil antes de procesar cada petición protegida.

### Stack de Seguridad
* **JWT (JSON Web Token):** Gestión de sesiones sin estado.
* **bcrypt:** Encriptación de contraseñas mediante hashing.
* **Express Middlewares:** Validación de token, roles y permisos en cada endpoint.
* **Seguridad en Tokens:** En el caso de usar cookies, se configurarán con las directivas `HttpOnly`, `Secure` y `SameSite`.

---

#### 2. APIs Principales (Endpoints)

| Grupo | Ruta (Ejemplo) | Función |
| :--- | :--- | :--- |
| **Autenticación** | `POST /api/auth/register`<br>`POST /api/auth/login` | Registro de adultos e inicio de sesión. |
| **Perfiles** | `GET /api/profiles/me`<br>`POST /api/profiles/children` | Consulta de perfil propio y creación de perfiles infantiles. |
| **Vinculación** | `POST /api/guardians/link` | Enlace entre un perfil adulto y un perfil infantil. |
| **Glucosa** | `POST /api/glucose`<br>`GET /api/glucose` | Registro y consulta del historial de glucosa según permisos. |
| **Comidas** | `POST /api/meals`<br>`GET /api/meals` | Registro y consulta de comidas e hidratos. |
| **Calculadora** | `GET /api/calculator/settings`<br>`PUT /api/calculator/settings` | Consulta (Niño/Adulto) o modificación (Solo Adulto) de parámetros. |
| **Recordatorios** | `GET /api/reminders`<br>`PUT /api/reminders` | Consulta y configuración de horarios de alertas. |

---

#### 3. APIs y Servicios Externos

Para acotar el alcance de la versión inicial y reducir la complejidad, se limita la dependencia de servicios de terceros:

| Servicio Externo | Uso Previsto | Límite Gratuito |
| :--- | :--- | :--- |
| **Sensores / Bombas de insulina** | **No se integra.** Registro 100% manual. | N/A |
| **Servicio de Email / Push / SMS** | **No se utiliza inicialmente.** Alertas in-app. | N/A |
| **Base de Datos (MongoDB Atlas)** | Persistencia de usuarios, registros y configuraciones. | Según plan M0 (512 MB gratis). |

### Base de datos (MongoDB)
Utilizaremos **MongoDB Atlas** (NoSQL) para almacenar los usuarios, perfiles infantiles, registros y configuraciones de la aplicación en documentos BSON organizados en colecciones.

---

#### Colecciones Principales

##### 1. `users`
Almacena los datos de autenticación y el rol de cada usuario registrado.

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `_id` | `ObjectId` | Identificador único del usuario. |
| `name` | `String` | Nombre del usuario. |
| `email` | `String` | Correo electrónico para iniciar sesión (único). |
| `passwordHash` | `String` | Hash de la contraseña (generado con `bcrypt`). |
| `role` | `String` | Rol del usuario (`child` o `guardian`). |
| `createdAt` | `Date` | Fecha de creación de la cuenta. |

---

##### 2. `childProfiles`
Contiene la información específica del perfil infantil, separada de las credenciales de acceso.

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `_id` | `ObjectId` | Identificador del perfil infantil. |
| `userId` | `ObjectId` | Referencia al usuario correspondiente en `users`. |
| `guardianIds` | `Array<ObjectId>` | Referencias a los usuarios adultos autorizados (`users`). |
| `nickname` | `String` | Nombre o apodo mostrado dentro de la app. |
| `birthDate` | `Date` | Fecha de nacimiento. |
| `createdAt` | `Date` | Fecha de creación del perfil. |

---

##### 3. `glucoseRecords`
Registra las lecturas de glucosa introducidas por el niño.

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `_id` | `ObjectId` | Identificador único del registro. |
| `childId` | `ObjectId` | Referencia al perfil infantil en `childProfiles`. |
| `value` | `Number` | Valor de la glucosa medida (ej. 110). |
| `unit` | `String` | Unidad de medida (ej. `mg/dL`). |
| `measuredAt` | `Date` | Fecha y hora en que se realizó la medición. |
| `createdAt` | `Date` | Fecha de creación del registro en el sistema. |

---

##### 4. `meals`
Almacena las comidas e hidratos de carbono registrados por el menor.

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `_id` | `ObjectId` | Identificador de la comida. |
| `childId` | `ObjectId` | Referencia al perfil infantil en `childProfiles`. |
| `description` | `String` | Descripción de la comida (ej. manzana, bocadillo). |
| `carbohydrates` | `Number` | Cantidad estimada de carbohidratos (en gramos). |
| `consumedAt` | `Date` | Fecha y hora de consumo. |
| `createdAt` | `Date` | Fecha de registro en el sistema. |

---

##### 5. `calculatorSettings`
Guarda los parámetros de la calculadora configurados exclusivamente por los tutores.

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `_id` | `ObjectId` | Identificador de la configuración. |
| `childId` | `ObjectId` | Referencia al perfil infantil en `childProfiles`. |
| `parameters` | `Object` | Objeto con parámetros educativos para la simulación (ratios, FCF, etc.). |
| `updatedBy` | `ObjectId` | Referencia al usuario adulto (`users`) que hizo la modificación. |
| `updatedAt` | `Date` | Fecha de la última actualización. |

---

##### 6. `reminders`
Almacena las preferencias y alertas de recordatorio personalizadas.

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `_id` | `ObjectId` | Identificador del recordatorio. |
| `childId` | `ObjectId` | Referencia al perfil infantil en `childProfiles`. |
| `createdBy` | `ObjectId` | Referencia al usuario adulto (`users`) que lo creó. |
| `time` | `String` | Hora programada (ej. `"08:00"`, `"14:30"`). |
| `daysOfWeek` | `Array<Number>` | Días activos (`1` = Lunes, `7` = Domingo). |
| `enabled` | `Boolean` | Estado del recordatorio (`true` / `false`). |
