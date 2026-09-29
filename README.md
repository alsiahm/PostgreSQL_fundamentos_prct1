**Administración y diseño de bases de datos**
*Práctica 1 - Conceptos fundamentales de PostgreSQL*
Alba Hidalgo Martín - alu0101619217

## 1. Creación de la base de datos

### 1.a. Crear una base de datos llamada biblioteca.

```sql
postgres=# CREATE DATABASE biblioteca;
CREATE DATABASE
```

Después nos conectamos a la base de datos:

```bash
psql -U postgres -d biblioteca
```

---

# 2. Creación de usuarios

### 2.a.i. Crear el usuario admin_biblio con permisos de administrador sobre la base de datos.

```sql
biblioteca=# CREATE USER admin_biblio WITH PASSWORD ' ';
CREATE ROLE

biblioteca=# GRANT ALL PRIVILEGES ON DATABASE biblioteca TO admin_biblio;
GRANT

biblioteca=# GRANT USAGE ON SCHEMA public TO admin_biblio;
GRANT
```
Le damos permisos en el esquema porque si no tampoco tendría permisos en la base de datos.

### 2.a.ii. Crear el usuario usuario_biblio con permisos solo de lectura.

```sql
biblioteca=# CREATE USER usuario_biblio WITH PASSWORD ' ';
CREATE ROLE
``` 
### 2.b. Crear un rol llamado lectores con permisos únicamente de consulta sobre todas las tablas de la base de datos (y asignarlo 2.c.).

```sql
biblioteca=# CREATE ROLE lectores;
CREATE ROLE

biblioteca=# GRANT CONNECT ON DATABASE biblioteca TO lectores;
GRANT

biblioteca=# GRANT lectores TO usuario_biblio;
GRANT ROLE

biblioteca=# GRANT USAGE ON SCHEMA public TO lectores;
GRANT
```

### 2.d. Consultar los roles existentes en PostgreSQL.

```sql
biblioteca=# SELECT rolname
FROM pg_roles;
```

```text
           rolname
-----------------------------
 pg_database_owner
 pg_read_all_data
 pg_write_all_data
 pg_monitor
 pg_read_all_settings
 pg_read_all_stats
 pg_stat_scan_tables
 pg_read_server_files
 pg_write_server_files
 pg_execute_server_program
 pg_signal_backend
 pg_checkpoint
 pg_use_reserved_connections
 pg_create_subscription
 postgres
 mydb_admin
 admin_biblio
 usuario_biblio
 lectores
(19 rows)
```

### 2.e. Cambiar la contraseña del usuario usuario_biblio.

```sql
biblioteca=# ALTER USER usuario_biblio
WITH PASSWORD 'new';
ALTER ROLE
```

### 2.f. Asegurar que usuario_biblio no pueda eliminar registros.

```sql
biblioteca=# REVOKE DELETE ON ALL TABLES IN SCHEMA public FROM lectores;
REVOKE
```

# 3. Creación de tablas

### 3.a. Crear las tablas.

```sql
biblioteca=# CREATE TABLE autores(
biblioteca(# id_autor SERIAL PRIMARY KEY,
biblioteca(# nombre VARCHAR(60) NOT NULL,
biblioteca(# nacionalidad VARCHAR(30));
CREATE TABLE
```

La columna nacionalidad puede quedar como `NULL` por si el autor es anónimo.

```sql
biblioteca=# CREATE TABLE libros(
biblioteca(# id_libro SERIAL PRIMARY KEY,
biblioteca(# titulo VARCHAR(100) NOT NULL,
biblioteca(# ano_publicacion INTEGER NOT NULL,
biblioteca(# id_autor INTEGER NOT NULL REFERENCES autores(id_autor) ON DELETE CASCADE);
CREATE TABLE
```

```sql
biblioteca=# CREATE TABLE prestamos(
id_prestamo SERIAL PRIMARY KEY,
id_libro INTEGER NOT NULL REFERENCES libros(id_libro) ON DELETE CASCADE,
fecha_prestamo DATE NOT NULL,
fecha_devolucion DATE,
usuario_prestatario VARCHAR(50) NOT NULL);
CREATE TABLE
```

# 4. Inserción de datos

```sql
biblioteca=# INSERT INTO autores (nombre, nacionalidad)
VALUES
('Jose Saramago', 'Portugal'),
('Unamuno', 'Espana'),
('Miguel Delibes', 'Espana'),
('Charles Dickens', 'Inglesa'),
('Natsume Soseki', 'Japon');
INSERT 0 5
```

```sql
biblioteca=# INSERT INTO libros (titulo, ano_publicacion, id_autor)
VALUES
('Memorial del convento', 1982, 1),
('Ensayo sobre la ceguera', 1995, 1),
('San Manuel Bueno, Martir', 1931, 2),
('Cinco horas con Mario', 1966, 3),
('El principe destronado', 1973, 3),
('Historia sobre 2 ciudades', 1859, 4),
('El almacen de antiguedades', 1840, 4),
('Las hierbas del Camino', 1915, 5);
INSERT 0 8
```

```sql
biblioteca=# INSERT INTO prestamos
(id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario)
VALUES
(1, '2025-08-19', '2026-08-19', 'Juan');
INSERT 0 1

biblioteca=# INSERT INTO prestamos
(id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario)
VALUES
(1, '2026-01-01', '2026-03-19', 'Elsa');
INSERT 0 1

biblioteca=# INSERT INTO prestamos
(id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario)
VALUES
(7, '2026-01-21', '2027-04-01', 'Pablo');
INSERT 0 1

biblioteca=# INSERT INTO prestamos
(id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario)
VALUES
(3, '2026-02-05', '2027-09-10', 'Pepe');
INSERT 0 1

biblioteca=# INSERT INTO prestamos
(id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario)
VALUES
(3, '2025-12-30', '2028-01-03', 'Elisa');
INSERT 0 1
```

# 5. Consultas sobre libros y autores

### 5.a. Listar los libros mostrando el autor correspondiente.

```sql
biblioteca=# SELECT titulo, nombre
FROM libros NATURAL JOIN autores;
```

```text
           titulo           |     nombre
----------------------------+-----------------
 Memorial del convento      | Jose Saramago
 Ensayo sobre la ceguera    | Jose Saramago
 San Manuel Bueno, Martir   | Unamuno
 Cinco horas con Mario      | Miguel Delibes
 El principe destronado     | Miguel Delibes
 Historia sobre 2 ciudades  | Charles Dickens
 El almacen de antiguedades | Charles Dickens
 Las hierbas del Camino     | Natsume Soseki
(8 rows)
```

### 5.b. Mostrar los préstamos que aún no tienen fecha de devolución.

```sql
biblioteca=# SELECT *
FROM prestamos
WHERE fecha_devolucion IS NULL;
```

```text
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
(0 rows)
```

### 5.c. Mostrar los autores que tienen más de un libro registrado.

```sql
biblioteca=# SELECT nombre
FROM autores NATURAL JOIN libros
GROUP BY nombre
HAVING COUNT(id_libro) > 1;
```

```text
     nombre
-----------------
 Jose Saramago
 Charles Dickens
 Miguel Delibes
(3 rows)
```

---

# 6. Consultas de agregación

### 6.a. Calcular el número total de préstamos realizados.

```sql
biblioteca=# SELECT COUNT(*)
FROM prestamos;
```

```text
 count
-------
     5
(1 row)
```

### 6.b. Obtener el número de libros prestados por cada usuario.

```sql
biblioteca=# SELECT usuario_prestatario, COUNT(*) AS libros
FROM prestamos
GROUP BY usuario_prestatario;
```

```text
 usuario_prestatario | libros
---------------------+--------
 Elsa                |      1
 Juan                |      1
 Elisa               |      1
 Pepe                |      1
 Pablo               |      1
(5 rows)
```

# 7. Modificación de datos

### 7.a. Actualizar la fecha de devolución de un préstamo pendiente.

```sql
biblioteca=# UPDATE prestamos
SET fecha_devolucion = '2026-10-7'
WHERE id_prestamo = 1;
UPDATE 1
```


### 7.b. Eliminar un libro y comprobar el efecto sobre los préstamos asociados.

Se comprueba antes de eliminar que el libro 1 tiene 2 préstamos asociados.

```sql
biblioteca=# SELECT *
FROM prestamos
WHERE id_libro = 1;
```

```text
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           2 |        1 | 2026-01-01     | 2026-03-19       | Elsa
           1 |        1 | 2025-08-19     | 2026-10-07       | Juan
(2 rows)
```

```sql
biblioteca=# DELETE FROM libros
WHERE id_libro = 1;
DELETE 1
```

```sql
biblioteca=# SELECT *
FROM prestamos
WHERE id_libro = 1;
```

```text
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
(0 rows)
```


# 8. Creación de vistas

### 8.a. Crear una vista llamada vista_libros_prestados que muestre: título del libro, autor y nombre del prestatario.

```sql
biblioteca=# CREATE VIEW vista_libros_prestados AS
SELECT titulo AS libro,
       nombre AS autor,
       usuario_prestatario
FROM prestamos
NATURAL JOIN libros
NATURAL JOIN autores;
CREATE VIEW
```

```sql
biblioteca=# SELECT *
biblioteca-# FROM vista_libros_prestados ;
```
```text
           libro            |      autor      | usuario_prestatario
----------------------------+-----------------+---------------------
 El almacen de antiguedades | Charles Dickens | Pablo
 San Manuel Bueno, Martir   | Unamuno         | Pepe
 San Manuel Bueno, Martir   | Unamuno         | Elisa
(3 rows)
```

### 8.b. Conceder permisos de consulta sobre esta vista únicamente a usuario_biblio.

```sql
biblioteca=# GRANT SELECT ON vista_libros_prestados TO usuario_biblio;
GRANT
```

# 9. Funciones y consultas avanzadas

### 9.a. Crear una función que reciba el nombre de un autor y devuelva todos sus libros.

```sql
biblioteca=# CREATE OR REPLACE FUNCTION libros_autor(nombre_autor VARCHAR)
RETURNS TABLE (
    id_libro INTEGER,
    titulo VARCHAR(100)
)
LANGUAGE SQL
AS $$
    SELECT id_libro, titulo
    FROM libros NATURAL JOIN autores
    WHERE nombre = nombre_autor;
$$;
CREATE FUNCTION

biblioteca=# SELECT * FROM libros_autor('Miguel Delibes');
```

```text
 id_libro |         titulo
----------+------------------------
        4 | Cinco horas con Mario
        5 | El principe destronado
(2 rows)
```




### 9.b. Mostrar los 3 libros que tienen mayor número de préstamos.

```sql
biblioteca=# SELECT titulo, COUNT(id_prestamo)
FROM libros NATURAL JOIN prestamos
GROUP BY id_libro, titulo
ORDER BY COUNT(id_prestamo) DESC
LIMIT 3;
```

Solo aparecen 2 libros con préstamos porque se eliminó anteriormente el libro con id_libro = 1. Debido a ON DELETE CASCADE, también se eliminaron sus dos préstamos.


```text
           titulo           | count
----------------------------+-------
 San Manuel Bueno, Martir   |     2
 El almacen de antiguedades |     1
(2 rows)
```

# 10. Exportación e importación de datos

### 10.a. Exportar la tabla libros a un archivo CSV.

```sql
biblioteca=# \copy libros TO '/home/usuario/libros.csv' WITH CSV HEADER;
COPY 7
```
Archivo incluido en el GitHub

### 10.b. Importar datos de editoriales desde un archivo CSV externo.

Creamos tabla que va a contener el csv (creada por nosotros auxiliarmente e incluida en el GitHub)

```sql
biblioteca=# CREATE TABLE autores_aux(
biblioteca(# id_autor SERIAL PRIMARY KEY, nombre VARCHAR(60) NOT NULL, nacionalidad VARCHAR(30) );
CREATE TABLE
biblioteca=# INSERT INTO autores_aux(nombre, nacionalidad)
biblioteca-# VALUES ('Isabel Allende', 'Chile'),
biblioteca-# ('Cervantes', 'Espana');
INSERT 0 2
```

Una vez simulado un CSV externo, realizamos la tarea de importarlo:

```sql
biblioteca-# biblioteca=# \copy autores(nombre, nacionalidad) FROM '/home/usuario/autores.csv' WITH CS
V HEADER;
```

Comprobamos los datos:

```sql
biblioteca=# SELECT *
FROM autores;
```

```text
 id_autor |     nombre      | nacionalidad
----------+-----------------+--------------
        1 | Jose Saramago   | Portugal
        2 | Unamuno         | Espana
        3 | Miguel Delibes  | Espana
        4 | Charles Dickens | Inglesa
        5 | Natsume Soseki  | Japon
        6 | Isabel Allende  | Chile
        7 | Cervantes       | Espana
(7 rows)
```

El resto de tablas resultan así:
```text
 id_libro |           titulo           | ano_publicacion | id_autor
----------+----------------------------+-----------------+----------
        2 | Ensayo sobre la ceguera    |            1995 |        1
        3 | San Manuel Bueno, Martir   |            1931 |        2
        4 | Cinco horas con Mario      |            1966 |        3
        5 | El principe destronado     |            1973 |        3
        6 | Historia sobre 2 ciudades  |            1859 |        4
        7 | El almacen de antiguedades |            1840 |        4
        8 | Las hierbas del Camino     |            1915 |        5
(7 rows) 

 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           3 |        7 | 2026-01-21     | 2027-04-01       | Pablo
           4 |        3 | 2026-02-05     | 2027-09-10       | Pepe
           5 |        3 | 2025-12-30     | 2028-01-03       | Elisa
(3 rows)
```