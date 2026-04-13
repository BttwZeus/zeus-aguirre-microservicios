# Proyecto de Microservicios - Zeus Aguirre

**Nombre:** Zeus Aguirre  
**Carrera:** Ingeniería de Software (6to Semestre)  
**Materia:** Diseño y Arquitectura de Software  
**Fecha:** Abril 2026

## ¿Qué problema resuelve mi aplicación?
Es un sistema de registro de usuarios diseñado para manejar altas cargas de trabajo. Separa el registro básico de las tareas pesadas (como notificaciones o procesamiento de datos) para que la página siempre responda rápido al cliente.

## ¿Cuál era el problema del monolito?
Cuando probé el monolito con el comando `ab` de Apache, los tiempos de respuesta estaban por los 730-740ms. Esto pasaba porque el servidor tenía que guardar al usuario en la base de datos y al mismo tiempo procesar una "notificación pesada". Si esa parte tardaba, toda la página se quedaba cargando y se saturaba fácil.

## ¿Qué responsabilidad tiene cada microservicio?
Separé el código en dos servicios independientes:
* **Servicio A:** Es el que ve el usuario. Se encarga de recibir los datos del formulario y guardarlos en la tabla principal.
* **Servicio B:** Es un worker que corre por detrás. Recibe avisos del Servicio A para hacer el "trabajo pesado" y guarda sus propios logs.

## ¿Cómo se comunican los servicios?
El Servicio A le avisa al Servicio B mediante una petición HTTP interna. En el código, en lugar de usar una IP, uso el nombre del contenedor `servicio_b` como hostname. Esto funciona gracias al DNS interno que crea la red de Docker Compose.

## Tablas en la base de datos (AWS RDS)
Usé MariaDB en AWS con una base de datos llamada `db_aguirre`. Así quedó la estructura:

| Tabla | Servicio dueño | Qué guarda |
|-------|---------------|------------|
| `usuarios_aguirre` | Servicio A | Datos del registro (nombre y correo) |
| `logs_procesamiento` | Servicio B | Logs de las notificaciones procesadas |

## ¿Qué pasa si el Servicio B se cae? (Resiliencia)
Esta es la mayor ventaja. Si apago el Servicio B, el Servicio A sigue funcionando perfectamente. El usuario puede registrarse y el sistema le avisa que el módulo de notificaciones está en mantenimiento, pero su registro **sí se guarda** en la DB. En el monolito, si fallaba el proceso de notificación, tronaba todo el registro.

## Cómo levantar el proyecto
1. Clonar este repositorio.
2. Entrar a la carpeta `microservicios`.
3. Revisar que el archivo `docker-compose.yml` tenga las credenciales correctas de RDS.
4. Ejecutar el comando:
```bash
sudo docker-compose up --build -d

Conclusión
Lo más difícil fue configurar el docker-compose.yml y las reglas de red en AWS, pero al final funcionó. Los microservicios son mejores para aplicaciones que necesitan estar siempre activas, aunque el código se vuelve un poco más complejo de conectar.
