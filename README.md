# Oracle Database Free en Ubuntu Linux usando Docker

Guía completa para instalar y configurar **Oracle Database Free** en **Ubuntu Linux** utilizando Docker, crear el contenedor, iniciar/detener la base de datos, conectarse desde DBeaver y dejar el entorno listo para comenzar a crear tablas y practicar SQL.

---

## 📋 Tabla de contenidos

* [1. Objetivo](#1-objetivo)
* [2. Arquitectura utilizada](#2-arquitectura-utilizada)
* [3. Requisitos](#3-requisitos)
* [4. Verificar Ubuntu](#4-verificar-ubuntu)
* [5. Instalar Docker](#5-instalar-docker)
* [6. Verificar Docker](#6-verificar-docker)
* [7. Descargar Oracle Database Free](#7-descargar-oracle-database-free)
* [8. Crear el contenedor Oracle](#8-crear-el-contenedor-oracle)
* [9. Verificar el contenedor](#9-verificar-el-contenedor)
* [10. Esperar a que Oracle termine de iniciar](#10-esperar-a-que-oracle-termine-de-iniciar)
* [11. Configurar reinicio automático](#11-configurar-reinicio-automático)
* [12. Instalar DBeaver](#12-instalar-dbeaver)
* [13. Configurar conexión Oracle en DBeaver](#13-configurar-conexión-oracle-en-dbeaver)
* [14. Probar la conexión](#14-probar-la-conexión)
* [15. Crear la primera tabla](#15-crear-la-primera-tabla)
* [16. Crear un usuario de práctica](#16-crear-un-usuario-de-práctica)
* [17. Comprobar las tablas](#17-comprobar-las-tablas)
* [18. Operaciones diarias del contenedor](#18-operaciones-diarias-del-contenedor)
* [19. Ver logs de Oracle](#19-ver-logs-de-oracle)
* [20. Solución de problemas](#20-solución-de-problemas)
* [21. Comandos de referencia rápida](#21-comandos-de-referencia-rápida)
* [22. Estructura final](#22-estructura-final)

---

# 1. Objetivo

El objetivo de este laboratorio es disponer de una instalación local de:

* Ubuntu Linux
* Docker
* Oracle Database Free
* Oracle Database dentro de un contenedor Docker
* DBeaver como cliente SQL
* Un usuario dedicado para practicar SQL

La arquitectura final será:

```text
┌──────────────────────────────────────┐
│          Ubuntu Linux                │
│                                      │
│  ┌────────────────────────────────┐  │
│  │            Docker              │  │
│  │                                │  │
│  │  ┌──────────────────────────┐  │  │
│  │  │      oracle-free         │  │  │
│  │  │                          │  │  │
│  │  │  Oracle Database Free    │  │  │
│  │  │                          │  │  │
│  │  │  Port: 1521              │  │  │
│  │  │  Service: FREEPDB1       │  │  │
│  │  └──────────────────────────┘  │  │
│  └────────────────────────────────┘  │
│                                      │
│              ↕ TCP 1521              │
│                                      │
│             DBeaver                  │
└──────────────────────────────────────┘
```

---

# 2. Arquitectura utilizada

La instalación utiliza Docker para evitar instalar Oracle directamente sobre el sistema operativo.

Esto permite:

* Mantener Oracle aislado del sistema.
* Iniciar y detener Oracle fácilmente.
* Eliminar el laboratorio sin afectar Ubuntu.
* Recrear el entorno cuando sea necesario.
* Practicar administración básica de Docker.
* Trabajar con Oracle SQL desde DBeaver.

La imagen utilizada es:

```text
gvenzl/oracle-free:latest
```

El contenedor se llamará:

```text
oracle-free
```

El puerto utilizado será:

```text
1521
```

El servicio Oracle será:

```text
FREEPDB1
```

---

# 3. Requisitos

Hardware recomendado:

* CPU x86_64
* 8 GB RAM mínimo
* 16 GB RAM recomendado
* 20 GB o más de espacio disponible

Software:

* Ubuntu Linux
* Docker
* DBeaver Community Edition

No es necesario instalar Oracle directamente en Ubuntu.

---

# 4. Verificar Ubuntu

Primero comprobamos la versión del sistema:

```bash
lsb_release -a
```

También podemos utilizar:

```bash
cat /etc/os-release
```

Comprobar arquitectura:

```bash
uname -m
```

Para un sistema x86_64 debería aparecer:

```text
x86_64
```

Comprobar memoria:

```bash
free -h
```

Comprobar espacio:

```bash
df -h
```

---

# 5. Instalar Docker

## 5.1 Actualizar los repositorios

```bash
sudo apt update
```

Actualizar paquetes:

```bash
sudo apt upgrade -y
```

---

## 5.2 Instalar Docker

En Ubuntu se puede instalar el paquete Docker disponible en los repositorios:

```bash
sudo apt install docker.io -y
```

---

## 5.3 Habilitar Docker

Hacer que Docker se inicie automáticamente con Ubuntu:

```bash
sudo systemctl enable docker
```

Iniciar Docker:

```bash
sudo systemctl start docker
```

Comprobar el servicio:

```bash
sudo systemctl status docker
```

Debe aparecer un estado similar a:

```text
Active: active (running)
```

Para salir de la pantalla de `status`:

```text
q
```

---

# 6. Verificar Docker

Comprobar la versión:

```bash
docker --version
```

Ejemplo:

```text
Docker version 29.x.x
```

También podemos probar Docker con:

```bash
sudo docker run hello-world
```

Si todo funciona correctamente, Docker mostrará un mensaje indicando que la instalación funciona.

Comprobar los contenedores:

```bash
sudo docker ps
```

Mostrar también contenedores detenidos:

```bash
sudo docker ps -a
```

---

# 7. Descargar Oracle Database Free

La imagen utilizada para este laboratorio es:

```text
gvenzl/oracle-free:latest
```

Docker puede descargarla automáticamente cuando creemos el contenedor.

También podemos descargarla previamente:

```bash
sudo docker pull gvenzl/oracle-free:latest
```

Ver las imágenes disponibles:

```bash
sudo docker images
```

Deberíamos encontrar algo similar a:

```text
gvenzl/oracle-free
```

---

# 8. Crear el contenedor Oracle

Ahora crearemos el contenedor.

```bash
sudo docker run -d \
  --name oracle-free \
  -p 1521:1521 \
  -e ORACLE_PASSWORD=Oracle123 \
  gvenzl/oracle-free:latest
```

## Explicación del comando

### `docker run`

Crea y ejecuta un nuevo contenedor.

---

### `-d`

Significa:

```text
detached
```

Ejecuta el contenedor en segundo plano.

---

### `--name oracle-free`

Define el nombre del contenedor:

```text
oracle-free
```

Esto permite utilizar posteriormente comandos sencillos como:

```bash
sudo docker start oracle-free
```

o:

```bash
sudo docker stop oracle-free
```

---

### `-p 1521:1521`

Mapea el puerto:

```text
Ubuntu:1521 → Docker:1521
```

El puerto `1521` es el puerto utilizado por Oracle Database para las conexiones SQL*Net/Oracle Net en este laboratorio.

---

### `-e ORACLE_PASSWORD=Oracle123`

Define la contraseña administrativa inicial de Oracle.

En este laboratorio:

```text
Password:
Oracle123
```

**Nota:** para un entorno real de producción debe utilizarse una contraseña fuerte y gestionada de forma segura. Esta contraseña se utiliza aquí únicamente como credencial de laboratorio.

---

### `gvenzl/oracle-free:latest`

Es la imagen de Oracle Database Free utilizada por el contenedor.

---

# 9. Verificar el contenedor

Después de crear el contenedor:

```bash
sudo docker ps
```

Deberíamos encontrar:

```text
oracle-free
```

con un estado parecido a:

```text
Up ...
```

También podemos consultar todos los contenedores:

```bash
sudo docker ps -a
```

Ejemplo:

```text
CONTAINER ID   IMAGE                        STATUS
xxxxxxxxxxxx   gvenzl/oracle-free:latest   Up ...
```

---

# 10. Esperar a que Oracle termine de iniciar

Crear el contenedor no significa que Oracle esté inmediatamente listo para recibir conexiones.

Oracle necesita unos segundos para inicializarse.

Podemos observar el proceso con:

```bash
sudo docker logs oracle-free
```

Para seguir los logs en tiempo real:

```bash
sudo docker logs -f oracle-free
```

Para salir:

```text
Ctrl + C
```

También podemos comprobar que el contenedor está ejecutándose:

```bash
sudo docker ps
```

---

# 11. Configurar reinicio automático

Una de las configuraciones recomendadas para este laboratorio es hacer que Docker vuelva a iniciar el contenedor automáticamente cuando el servicio Docker vuelva a estar disponible.

Ejecutar una sola vez:

```bash
sudo docker update --restart unless-stopped oracle-free
```

Comprobar la política:

```bash
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' oracle-free
```

Debe mostrar:

```text
unless-stopped
```

Esto significa que después de reiniciar Ubuntu, Docker podrá iniciar nuevamente el contenedor.

---

# 12. Instalar DBeaver

DBeaver será el cliente gráfico utilizado para trabajar con Oracle.

En Ubuntu podemos instalar DBeaver mediante el método disponible para nuestra distribución.

Una vez instalado, abrir:

```text
DBeaver
```

La idea es:

```text
DBeaver
   ↓
localhost:1521
   ↓
Docker
   ↓
oracle-free
   ↓
Oracle Database
```

---

# 13. Configurar conexión Oracle en DBeaver

Crear una nueva conexión:

```text
New Database Connection
```

Seleccionar:

```text
Oracle
```

Configurar:

| Parámetro        | Valor           |
| ---------------- | --------------- |
| Host             | `localhost`     |
| Port             | `1521`          |
| Database/Service | `FREEPDB1`      |
| Authentication   | Database Native |
| Username         | `franklyn`      |
| Password         | `Sql12345`      |

La conexión utilizada en este laboratorio es:

```text
Host: localhost
Port: 1521
Service Name: FREEPDB1
Username: franklyn
Password: Sql12345
```

> Estas credenciales corresponden al usuario de práctica creado para este laboratorio.

---

# 14. Probar la conexión

En DBeaver utilizar:

```text
Test Connection
```

Si todo está funcionando correctamente, DBeaver debería indicar que la conexión fue exitosa.

Después seleccionar:

```text
Finish
```

y abrir la conexión.

---

# 15. Crear la primera tabla

Una vez conectados a Oracle desde DBeaver, podemos crear nuestra primera tabla.

Ejemplo:

```sql
CREATE TABLE employees (
    employee_id NUMBER PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    salary NUMBER(10,2)
);
```

Ejecutar la consulta.

Comprobar la tabla:

```sql
SELECT table_name
FROM user_tables
ORDER BY table_name;
```

Debería aparecer:

```text
EMPLOYEES
```

---

# 16. Crear un usuario de práctica

Si queremos crear un usuario dedicado para nuestro laboratorio, debemos hacerlo desde una conexión con privilegios administrativos.

Conectarse como administrador y utilizar:

```sql
CREATE USER franklyn IDENTIFIED BY Sql12345;
```

Asignar permisos básicos:

```sql
GRANT CREATE SESSION TO franklyn;
```

Permitir creación de tablas:

```sql
GRANT CREATE TABLE TO franklyn;
```

Permitir crear vistas:

```sql
GRANT CREATE VIEW TO franklyn;
```

Permitir crear secuencias:

```sql
GRANT CREATE SEQUENCE TO franklyn;
```

Permitir crear procedimientos:

```sql
GRANT CREATE PROCEDURE TO franklyn;
```

Para un laboratorio también podemos asignar cuota en el tablespace:

```sql
ALTER USER franklyn QUOTA UNLIMITED ON USERS;
```

Después conectar DBeaver utilizando:

```text
Username: franklyn
Password: Sql12345
Service: FREEPDB1
```

> Si el usuario ya fue creado anteriormente, no es necesario ejecutar nuevamente `CREATE USER`.

---

# 17. Comprobar las tablas

Una vez conectados como `franklyn`, podemos consultar las tablas pertenecientes al usuario:

```sql
SELECT table_name
FROM user_tables
ORDER BY table_name;
```

También podemos consultar las columnas de una tabla:

```sql
SELECT column_name,
       data_type,
       data_length
FROM user_tab_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY column_id;
```

Otra forma rápida de inspeccionar una tabla desde SQL*Plus/SQLcl es:

```sql
DESC employees;
```

En DBeaver también podemos utilizar el navegador de objetos para inspeccionar:

```text
Tables
 └── EMPLOYEES
```

---

# 18. Operaciones diarias del contenedor

Una vez creado el contenedor, **no es necesario volver a ejecutar `docker run`** cada vez que queramos utilizar Oracle.

## Verificar si está funcionando

```bash
sudo docker ps
```

Si aparece:

```text
oracle-free ... Up ...
```

Oracle está ejecutándose.

---

## Iniciar Oracle

Si aparece como `Exited`:

```bash
sudo docker start oracle-free
```

Después:

```bash
sudo docker ps
```

---

## Detener Oracle

```bash
sudo docker stop oracle-free
```

---

## Reiniciar Oracle

```bash
sudo docker restart oracle-free
```

---

## Ver todos los contenedores

```bash
sudo docker ps -a
```

---

# 19. Ver logs de Oracle

Para consultar los logs:

```bash
sudo docker logs oracle-free
```

Para seguirlos en tiempo real:

```bash
sudo docker logs -f oracle-free
```

Los logs son especialmente útiles cuando:

* Oracle no inicia.
* DBeaver no puede conectarse.
* El contenedor aparece como `Exited`.
* Oracle todavía está inicializando.
* Hay algún problema con la configuración.

---

# 20. Solución de problemas

## Problema 1 — `oracle-free` aparece como `Exited`

Comprobar:

```bash
sudo docker ps -a
```

Si aparece:

```text
oracle-free   Exited
```

intentar:

```bash
sudo docker start oracle-free
```

Después:

```bash
sudo docker ps
```

Si vuelve a detenerse:

```bash
sudo docker logs oracle-free
```

---

## Problema 2 — DBeaver no conecta

Comprobar primero Docker:

```bash
sudo docker ps
```

Debe aparecer:

```text
oracle-free
```

Comprobar el puerto:

```bash
sudo docker port oracle-free
```

Esperamos algo parecido a:

```text
1521/tcp -> 0.0.0.0:1521
```

Comprobar los logs:

```bash
sudo docker logs oracle-free
```

Revisar en DBeaver:

```text
Host: localhost
Port: 1521
Service: FREEPDB1
```

---

## Problema 3 — El puerto 1521 ya está ocupado

Comprobar:

```bash
sudo ss -ltnp | grep 1521
```

Si otro proceso está utilizando el puerto, habrá que identificarlo antes de crear otro mapeo.

---

## Problema 4 — El contenedor no existe

Comprobar:

```bash
sudo docker ps -a
```

Si `oracle-free` no aparece, entonces el contenedor no existe en esa instalación de Docker.

En ese caso se debe crear nuevamente:

```bash
sudo docker run -d \
  --name oracle-free \
  -p 1521:1521 \
  -e ORACLE_PASSWORD=Oracle123 \
  gvenzl/oracle-free:latest
```

---

# 21. Comandos de referencia rápida

## Docker

```bash
sudo docker ps
```

Ver contenedores activos.

```bash
sudo docker ps -a
```

Ver todos los contenedores.

```bash
sudo docker start oracle-free
```

Iniciar Oracle.

```bash
sudo docker stop oracle-free
```

Detener Oracle.

```bash
sudo docker restart oracle-free
```

Reiniciar Oracle.

```bash
sudo docker logs oracle-free
```

Ver logs.

```bash
sudo docker logs -f oracle-free
```

Seguir logs en tiempo real.

```bash
sudo docker inspect oracle-free
```

Ver información detallada del contenedor.

```bash
sudo docker port oracle-free
```

Ver puertos publicados.

---

# 22. Comandos SQL iniciales

Comprobar usuario:

```sql
SELECT USER
FROM dual;
```

Comprobar fecha del servidor:

```sql
SELECT SYSDATE
FROM dual;
```

Ver tablas del usuario:

```sql
SELECT table_name
FROM user_tables
ORDER BY table_name;
```

Crear tabla:

```sql
CREATE TABLE test_table (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(100)
);
```

Insertar datos:

```sql
INSERT INTO test_table (id, name)
VALUES (1, 'Franklyn');
```

Confirmar:

```sql
COMMIT;
```

Consultar:

```sql
SELECT *
FROM test_table;
```

Eliminar tabla de prueba:

```sql
DROP TABLE test_table;
```

---

# 23. Flujo de trabajo recomendado

Una vez configurado el laboratorio, el flujo normal será:

```text
Ubuntu
  │
  ▼
Docker
  │
  ▼
oracle-free
  │
  ▼
Oracle Database
  │
  ▼
FREEPDB1
  │
  ▼
DBeaver
  │
  ▼
SQL
```

Cuando vuelvas a Ubuntu:

### 1. Comprobar Oracle

```bash
sudo docker ps
```

### 2. Si está detenido

```bash
sudo docker start oracle-free
```

### 3. Abrir DBeaver

Conectar utilizando:

```text
Host: localhost
Port: 1521
Service: FREEPDB1
User: franklyn
```

### 4. Empezar a trabajar

Ejemplo:

```sql
SELECT *
FROM employees;
```

---

# 24. Crear el entorno completo de práctica

Una vez que Oracle está funcionando, podemos crear una estructura de laboratorio más completa.

Ejemplo:

```text
Oracle Database
│
└── FREEPDB1
    │
    └── FRANKLYN
        │
        ├── DEPARTMENTS
        ├── EMPLOYEES
        ├── PROJECTS
        ├── EMPLOYEE_PROJECTS
        ├── CUSTOMERS
        └── ORDERS
```

Estas tablas permiten practicar:

* SELECT
* WHERE
* ORDER BY
* DISTINCT
* LIKE
* IN
* BETWEEN
* IS NULL
* COUNT
* SUM
* AVG
* MIN
* MAX
* GROUP BY
* HAVING
* INNER JOIN
* LEFT JOIN
* SELF JOIN
* CASE
* NVL
* funciones de texto
* funciones de fecha
* subconsultas
* EXISTS
* NOT EXISTS
* UNION
* INSERT
* UPDATE
* DELETE
* COMMIT
* ROLLBACK

---

# 25. Buenas prácticas

## No ejecutar `docker run` cada vez

`docker run` crea un contenedor nuevo.

Después de crear:

```text
oracle-free
```

utiliza:

```bash
sudo docker start oracle-free
```

para iniciarlo.

---

## Verificar antes de trabajar

```bash
sudo docker ps
```

Es una buena práctica comprobar el estado antes de abrir DBeaver.

---

## No modificar datos sin WHERE

Evitar:

```sql
UPDATE employees
SET salary = salary * 1.10;
```

durante las prácticas, a menos que realmente quieras modificar todos los empleados.

Primero prueba:

```sql
SELECT *
FROM employees
WHERE employee_id = 10;
```

Después:

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE employee_id = 10;
```

---

## Probar DELETE con SELECT

Antes de:

```sql
DELETE FROM employees
WHERE ...;
```

hacer:

```sql
SELECT *
FROM employees
WHERE ...;
```

Así puedes verificar exactamente qué filas serán eliminadas.

---

# 26. Estado final del laboratorio

Al terminar la configuración tendremos:

```text
Ubuntu Linux
    │
    ├── Docker
    │
    └── Oracle Database Free
            │
            ├── Container: oracle-free
            ├── Port: 1521
            └── Service: FREEPDB1
                    │
                    └── User: franklyn
                            │
                            └── DBeaver
```

La configuración queda lista para comenzar a trabajar con SQL.

---

# 27. Comprobación final

Ejecutar:

```bash
sudo docker ps
```

Esperamos:

```text
oracle-free
```

con estado:

```text
Up
```

Después conectarse desde DBeaver y ejecutar:

```sql
SELECT USER,
       SYSDATE
FROM dual;
```

Si devuelve el usuario y la fecha/hora, la conexión Oracle está funcionando.

Comprobar tablas:

```sql
SELECT table_name
FROM user_tables
ORDER BY table_name;
```

A partir de este punto, el entorno está preparado para crear tablas y comenzar el entrenamiento de SQL.

---

# 🚀 Próximo paso

Después de completar esta configuración, el siguiente paso es crear el **dataset de práctica** y comenzar con consultas SQL progresivamente:

```text
SELECT
   ↓
WHERE
   ↓
ORDER BY
   ↓
GROUP BY
   ↓
HAVING
   ↓
INNER JOIN
   ↓
LEFT JOIN
   ↓
SELF JOIN
   ↓
SUBQUERIES
   ↓
EXISTS
   ↓
CASE / NVL
   ↓
FECHAS
   ↓
UNION
   ↓
DML
   ↓
CONSULTAS DE ENTREVISTA
```

La meta no es solamente memorizar comandos, sino poder recibir una pregunta como:

> "Obtén los departamentos de Santo Domingo que tengan más de 5 empleados y muestra su salario promedio."

y convertirla paso a paso en una consulta SQL correcta.

---

## 📝 Notas

Este repositorio está pensado como laboratorio educativo para practicar:

* Linux
* Docker
* Oracle Database
* DBeaver
* SQL
* Administración básica de bases de datos

No utilizar las credenciales incluidas en este README para un entorno de producción.

Para producción se deben utilizar credenciales seguras, secretos gestionados apropiadamente, restricciones de privilegios, políticas de acceso y una configuración de seguridad adecuada.
