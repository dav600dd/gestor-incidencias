# Gestor de incidencias con Docker Compose

Despliegue de la aplicación de incidencias con tres servicios (frontend con Nginx, backend con la API y PostgreSQL), dos redes (`frontend` y `datos`) y un volumen (`postgres_data`).

## Cómo arrancarlo

1. Copiar `.env.sample` a `.env` y ajustar los valores (sobre todo la contraseña).
2. Levantar todo con `docker compose up -d`.
3. Abrir `http://localhost:8080` (o el puerto que se haya puesto en `PUERTO_WEB`).

Para parar sin perder datos: `docker compose down`. Para borrar también los datos: `docker compose down -v` (cuidado).

## Preguntas

**1. ¿Por qué el frontend no necesita conectarse a PostgreSQL?**
Porque el navegador solo habla con el frontend y este reenvía las peticiones `/api/` al backend. Quien consulta y modifica los datos es la API, así que el frontend no tiene ningún motivo para llegar a la base de datos.

**2. ¿Por qué el backend pertenece a dos redes?**
Porque es el intermediario: tiene que recibir las peticiones del frontend (red `frontend`) y a la vez conectarse con PostgreSQL (red `datos`). Si estuviera en una sola, le faltaría acceso a uno de los dos lados.

**3. ¿Qué servicio necesita publicar un puerto en el anfitrión?**
Solo el frontend, porque es lo que usa la persona desde el navegador. El backend y la base de datos se comunican por las redes internas y no hace falta exponerlos.

**4. ¿Por qué publicar PostgreSQL hacia fuera no es la solución adecuada para un fallo de comunicación interno?**
Porque el problema estaría dentro de las redes de Docker (una red mal asignada, un nombre mal escrito, etc.) y publicar el puerto no lo arregla, solo lo oculta. Además deja la base de datos accesible desde fuera, lo cual es un riesgo de seguridad. Lo correcto es revisar las redes, los nombres de servicio y los logs.

**5. ¿Qué ventaja ofrece Docker Compose respecto a ejecutar los tres contenedores manualmente?**
Todo queda descrito en un único archivo y se levanta con un comando, en vez de repetir muchos `docker run`, `docker network create` y `docker volume create`. Es más fácil de repetir, de revisar y de compartir, y se evitan errores al configurar cada recurso a mano.

**6. ¿Por qué hemos dividido la arquitectura en dos redes?**
Para aislar los servicios y que cada uno vea solo lo que necesita. El frontend no puede acceder a la base de datos aunque quisiera, y eso reduce el daño posible si alguno de los servicios tuviera un problema o fuera comprometido.

**7. ¿Por qué el backend utiliza `db` como dirección de PostgreSQL?**
Porque `db` es el nombre del servicio en el Compose y Docker lo resuelve por DNS dentro de la red compartida. Usar `localhost` no funcionaría, ya que dentro del contenedor del backend `localhost` es el propio backend. Tampoco conviene usar la IP, porque cambia al recrear los contenedores.

**8. ¿Qué diferencia existe entre detener un proyecto y eliminar sus volúmenes?**
Detener (`stop`) o hacer `down` solo para o elimina los contenedores y las redes, pero el volumen se conserva y los datos siguen ahí al volver a arrancar. Con `down -v` se borra también el volumen y se pierden los datos de la base de datos.

**9. ¿Qué consecuencias tendría para la empresa perder el volumen de PostgreSQL?**
Se perderían todas las incidencias registradas, es decir, el historial de problemas y su seguimiento. La aplicación seguiría arrancando pero vacía, y habría que recuperar la información desde una copia de seguridad. Por eso hay que tener cuidado con `down -v` y hacer copias del volumen.
