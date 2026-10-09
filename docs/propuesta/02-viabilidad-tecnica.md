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

