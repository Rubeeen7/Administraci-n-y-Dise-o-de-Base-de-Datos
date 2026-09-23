# Práctica 1 - Conceptos fundamentales de PostgreSQL

## 1. Creación de la base de datos

Se crea una base de datos llamada `biblioteca`.

### Comando empleado

```sql
CREATE DATABASE biblioteca;
```

### Salida obtenida

```text
CREATE DATABASE
```

### Comprobación

Para comprobar que la base de datos se ha creado correctamente se ejecuta:

```text
\l
```

En la salida aparece la base de datos `biblioteca`.

## 2. Creación de usuarios

### 2.a Creación de los usuarios

Se crean los usuarios `admin_biblio` y `usuario_biblio`.

### Comandos empleados

```sql
CREATE USER admin_biblio WITH PASSWORD 'admin123';

CREATE USER usuario_biblio WITH PASSWORD 'usuario123';
```

### Salida obtenida

```text
CREATE ROLE
CREATE ROLE
```

### Comprobación

Para comprobar que los usuarios se han creado correctamente, se consulta la tabla del sistema `pg_roles`:

```sql
SELECT rolname
FROM pg_roles
WHERE rolname IN ('admin_biblio', 'usuario_biblio');
```

### Salida obtenida

```text
     rolname
-----------------
 admin_biblio
 usuario_biblio
(2 rows)
```
