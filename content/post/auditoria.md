---
title: "Auditoría"
date: 2023-02-20T11:31:49+01:00
draft: true
tags: ["bbdd"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

## 1. Activa desde SQL*Plus la auditoría de los intentos de acceso exitosos al sistema. Comprueba su funcionamiento.

Podemos ver los parámetros de auditoria con `show parameter audit` o haciendo una consulta a v$parameter.

```sql
show parameter audit

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /opt/oracle/admin/ORCLCDB/adump
audit_syslog_level                   string
audit_sys_operations                boolean      TRUE
audit_trail                          string      DB
unified_audit_common_systemlog       string
unified_audit_sga_queue_size         integer     1048576
unified_audit_systemlog              string
```

```sql
SELECT name, value FROM v$parameter WHERE name like 'audit_trail';

NAME
--------------------------------------------------------------------------------
VALUE
--------------------------------------------------------------------------------
audit_trail
DB
```

Vemos que el parámetro audit_trail está activado ya que su valor es 'DB'. DB significa que se activa la auditoría y los datos se almacenarán en la tabla SYS.AUD$ de Oracle (es equivalente a TRUE).

Si estuviera desactivado su valor sería 'NONE' y lo activaríamos con el siguiente comando:

```sql
ALTER SYSTEM SET audit_trail=db scope=spfile;
```

>Tras activarlo habría que reiniciar la base de datos de Oracle haciendo `shutdown` y `startup`.

Cuando el valor del parámetro audit_trail sea 'DB' ya podremos activar la auditoría de intentos de accesos. Para auditar tanto los intentos fallidos como los exitosos se utiliza:

```sql
AUDIT SESSION;
```

Para auditar solamente los fallidos:

```sql
AUDIT CREATE SESSION WHENEVER NOT SUCCESSFUL;
```

Y para auditar los exitosos:

```sql
AUDIT CREATE SESSION WHENEVER SUCCESSFUL;
```

En mi caso voy a auditar tanto los fallidos como los exitosos.

Ahora intentamos acceder con dos usuarios que no existen y uno que sí existe.

![accesos](/img/audit/1.png)

Comprobamos los intentos de acceso.

```sql
SELECT OS_USERNAME, USERNAME, EXTENDED_TIMESTAMP, ACTION_NAME, RETURNCODE FROM DBA_AUDIT_SESSION;


OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
--------------------------------------------------------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
ACTION_NAME                  RETURNCODE
---------------------------- ----------
usuario
ADMIN
20/02/23 12:38:28,851148 +01:00
LOGON                             0


OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
--------------------------------------------------------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
ACTION_NAME                  RETURNCODE
---------------------------- ----------
usuario
PEPITO
20/02/23 12:09:55,805600 +01:00
LOGON                          1017


OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
--------------------------------------------------------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
ACTION_NAME                   RETURNCODE
---------------------------- ----------
usuario
BASILISCO
20/02/23 12:10:04,099451 +01:00
LOGON                          1017
```

Si queremos ver las auditorías activadas:

```sql
SELECT * FROM dba_priv_audit_opts;


USER_NAME
--------------------------------------------------------------------------------
PROXY_NAME
--------------------------------------------------------------------------------
PRIVILEGE                                SUCCESS    FAILURE
---------------------------------------- ---------- ----------


CREATE SESSION                           BY ACCESS    BY ACCESS
```

Y si quisiéramos desactivar la auditoría que hemos activado en este ejercicio:

```sql
NOAUDIT SESSION;
-- ó:
NOAUDIT CREATE SESSION WHENEVER NOT SUCCESSFUL;
-- ó:
NOAUDIT CREATE SESSION WHENEVER SUCCESSFUL;
```


## 2. Realiza un procedimiento en PL/SQL que te muestre los accesos fallidos junto con el motivo de los mismos, transformando el código de error almacenado en un mensaje de texto comprensible. Contempla todos los motivos posibles para que un acceso sea fallido.

Primero he creado una función que explique el código almacenado.

```sql
CREATE OR REPLACE FUNCTION CodigoExplicado (p_cod DBA_AUDIT_SESSION.RETURNCODE%TYPE)
RETURN VARCHAR2
IS
  v_resultado VARCHAR2(4000);
BEGIN
  CASE p_cod
    WHEN 0 THEN
      v_resultado := 'Acceso correcto';
    WHEN 1004 THEN
      v_resultado := 'Acceso denegado. Nombre de usuario por defecto no compatible';
    WHEN 1017 THEN
      v_resultado := 'El usuario y/o contrasena es invalido';
    WHEN 1045 THEN
      v_resultado := 'El usuario no tiene el privilegio CREATE SESSION';
    WHEN 28000 THEN
      v_resultado := 'La cuenta esta bloqueada';
    WHEN 28001 THEN
      v_resultado := 'La contrasena ha expirado';
    WHEN 28002 THEN
      v_resultado := 'La contrasena vencerá en pocos dias';
    WHEN 28003 THEN
      v_resultado := 'La contrasena no es lo bastante compleja';
    WHEN 28007 THEN
      v_resultado := 'No puedes reutilizar la contrasena';
    WHEN 28008 THEN
      v_resultado := 'Has utilizado un contrasena antigua inválida';
    WHEN 28009 THEN
      v_resultado := 'La conexión como SYS debe ser a traves de SYSDBA o SYSOPER';
    WHEN 28011 THEN
      v_resultado := 'La cuenta expirara pronto, cambia la contrasena ahora';
    ELSE
      v_resultado := 'Ha habido un ERROR. Pongase en contacto con el administrador para saber mas';
  END CASE;

  RETURN v_resultado;
END CodigoExplicado;
/
```

Creo el procedimiento que muestre los accesos fallidos. Como en mi caso se almacenaban tanto los intentos exitosos como fallidos cuando haya un acceso con código 0 (acceso correcto) no se mostrará nada.

```sql
CREATE OR REPLACE PROCEDURE MostrarAccesosFallidos 
IS
    CURSOR C_INTENTOS_FALLIDOS IS 
        SELECT OS_USERNAME, USERNAME, EXTENDED_TIMESTAMP, RETURNCODE FROM DBA_AUDIT_SESSION;
    V_REG C_INTENTOS_FALLIDOS%ROWTYPE;
BEGIN
    DBMS_OUTPUT.PUT_LINE('--ACCESOS FALLIDOS--'||CHR(10)||'-------------------------------------------------------');
    FOR V_REG IN C_INTENTOS_FALLIDOS LOOP
        --Si es diferente a 0 (Acceso correcto) significa que ha habido un error por lo que se muestran los datos importantes del intento de acceso y la explicación del código usando la funcion CodigoExplicado
        IF CodigoExplicado(V_REG.RETURNCODE)!='Acceso correcto' THEN
            DBMS_OUTPUT.PUT_LINE('Usuario del sistema: '||V_REG.OS_USERNAME||CHR(10)||'Usuario de la base de datos: '||V_REG.USERNAME||CHR(10)||'Fecha y Hora: '||V_REG.EXTENDED_TIMESTAMP||CHR(10)||'Motivo: '||CodigoExplicado(V_REG.RETURNCODE)||CHR(10)||'-------------------------------------------------------');
        END IF;
    END LOOP;
END MostrarAccesosFallidos;
/
```

**Comprobación:**

```sql
exec MostrarAccesosFallidos;
```

![errores](/img/audit/2.png)


## 3. Activa la auditoría de las operaciones DML realizadas por SCOTT. Comprueba su funcionamiento.

Para activar la auditoría que se nos pide usamos el siguiente comando:

```sql
AUDIT INSERT TABLE, UPDATE TABLE, DELETE TABLE BY SCOTT;
```

>BY ACCESS realiza un registro por cada acción. BY SESSION realizaría un registro de todas las acciones por cada sesión iniciada.

Pobamos haciendo insert, update y delete con el usuario Scott para que se auditen los cambios.

```sql
INSERT INTO dept VALUES(50,'RECEPTION','SAN FRANCISCO');
UPDATE dept SET loc='WASHINGTON' WHERE deptno=50;
DELETE FROM dept WHERE deptno=50;
COMMIT;
```

**Comprobación:**

```sql
SELECT os_username, username, obj_name, action_name, extended_timestamp
FROM dba_audit_object
WHERE username='SCOTT';
```

Resultados:

```sql
OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
--------------------------------------------------------------------------------
OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME
----------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
usuario
SCOTT
DEPT
INSERT
20/02/23 14:34:28,780958 +01:00


OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
--------------------------------------------------------------------------------
OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME
----------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
usuario
SCOTT
DEPT
UPDATE
20/02/23 14:34:28,783132 +01:00


OS_USERNAME
--------------------------------------------------------------------------------
USERNAME
--------------------------------------------------------------------------------
OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME
----------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
usuario
SCOTT
DEPT
DELETE
20/02/23 14:34:28,787416 +01:00
```

## 4. Realiza una auditoría de grano fino para almacenar información sobre la inserción de empleados con sueldo superior a 2000 en la tabla emp de scott.

En la Auditoría de Grano Fino se guarda qué consulta ejecutó un usuario sobre una tabla en un momento determinado o qué datos fueron borrados, modificados o insertados por parte del usuario.

Vamos a crear una auditoría de grano fino que guardará información sobre la inserción de nuevos empleados con un sueldo superior a 2000 en la tabla emp de Scott. Para ello realizaremos un procedimiento con la siguiente política desde SYS:

```sql
BEGIN
    DBMS_FGA.ADD_POLICY (
        object_schema      =>  'SCOTT',
        object_name        =>  'EMP',
        policy_name        =>  'pol_nomayor2000',
        audit_condition    =>  'SAL > 2000',
        statement_types    =>  'INSERT'
    );
END;
/
```

Podemos ver la política creada de la siguiente forma:

```sql
select object_schema,object_name,policy_name,policy_text
from dba_audit_policies;


OBJECT_SCHEMA
--------------------------------------------------------------------------------
OBJECT_NAME
--------------------------------------------------------------------------------
POLICY_NAME
--------------------------------------------------------------------------------
POLICY_TEXT
--------------------------------------------------------------------------------
SCOTT
EMP
POL_NOMAYOR2000
SAL > 2000
```

Ahora con el usuario Scott insertamos algunos registros con sueldosuperior a 2000 para comprobar los datos guardados en la auditoría.

```sql
insert into emp values(8000,'PEPITO','CLERK',null,sysdate,3000,0,20);
insert into emp values(8001,'WILSON','CLERK',null,sysdate,1000,0,20);
insert into emp values(8002,'RUBY','CLERK',null,sysdate,2500,0,20);
insert into emp values(8003,'RICOH','CLERK',null,sysdate,800,0,20);

commit;
```

Comprobamos los datos insertados desde SYS:

```sql
SELECT db_user, object_name, sql_text, extended_timestamp
FROM dba_fga_audit_trail
WHERE policy_name='POL_NOMAYOR2000';
```

Nos aparecerá la siguiente información:

```sql
DB_USER
--------------------------------------------------------------------------------
OBJECT_NAME
--------------------------------------------------------------------------------
SQL_TEXT
--------------------------------------------------------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
SCOTT
EMP
insert into emp values(8000,'PEPITO','CLERK',null,sysdate,3000,0,20)
20/02/23 18:07:35,471112 +01:00


DB_USER
--------------------------------------------------------------------------------
OBJECT_NAME
--------------------------------------------------------------------------------
SQL_TEXT
--------------------------------------------------------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
SCOTT
EMP
insert into emp values(8002,'RUBY','CLERK',null,sysdate,2500,0,20)
20/02/23 18:07:35,488884 +01:00
```

Vemos que solo nos aparecen dos de los registros ya que solo dos de ellos tienen el salario superior a 2000.

## 5. Explica la diferencia entre auditar una operación by access o by session ilustrándolo con ejemplos.

La diferenca entre una y otra está en cómo se registra la información de auditoría.

La auditoría BY ACCESS crea una entrada en el registro de auditoría para cada operación de acceso a los recursos. Esto significa que cada vez que un usuario intenta acceder al recurso, se registra una entrada en el registro de auditoría, incluso si el usuario no tiene éxito.

Por otro lado, la auditoría BY SESSION registra una entrada en el registro de auditoría para cada sesión de usuario. Esto significa que la entrada se registra solo cuando el usuario inicia una sesión en el recurso, y no cada vez que el usuario intenta acceder a dicho recurso.

En resumen, la auditoría BY ACCESS es más detallada y puede generar una gran cantidad de registros, mientras que la auditoría BY SESSION es menos detallada pero más fácil de analizar y entender en términos de sesiones de usuario.

La forma de utilizar cada uno de ellos sería añadiéndolo al final de la sentencia durante la creción de una auditoría. Por ejemplo en la auditaría del ejercicio 3 sería de la siguiente forma:

```sql
AUDIT INSERT TABLE, UPDATE TABLE, DELETE TABLE BY SCOTT BY ACCESS;
-- ó:
AUDIT INSERT TABLE, UPDATE TABLE, DELETE TABLE BY SCOTT BY SESSION;
```

## 6. Documenta las diferencias entre los valores 'DB' y 'DB,EXTENDED' del parámetro audit_trail de ORACLE. Demuéstralas poniendo un ejemplo de la información sobre una operación concreta recopilada con cada uno de ellos.

El parámetro audit_trail en Oracle determina el nivel de detalle de la información de auditoría que se va a registrar. Hay diferentes valores posibles para este parámetro, como 'OS', 'DB', 'DB,EXTENDED', 'XML' y 'XML,EXTENDED'. En este caso vamos a ver 'DB' y 'DB,EXTENDED'.

- DB: con este valor, Oracle registra la información de auditoría en la tabla SYS.AUD$, que incluye el nombre del usuario, la fecha y hora, el comando ejecutado y el objeto afectado por el comando. Este nivel de auditoría proporciona información básica sobre los comandos ejecutados.

- DB,EXTENDED: con este valor, Oracle registra información adicional en la tabla SYS.AUD$. Guardaría todos los datos como en 'DB' y además escribe los valores correspondientes en las columnas SQLBIND y SQLTEXT de la misma tabla.

Los registros almacenados en la tabla SYS.AUD$ se pueden ver directamente en ésta o en las siguientes vistas:

```sql
SELECT view_name
FROM   dba_views
WHERE  view_name LIKE 'DBA%AUDIT%'
ORDER BY view_name;
```

```sql
VIEW_NAME
--------------------------------------------------------------------------------
DBA_AUDIT_EXISTS
DBA_AUDIT_MGMT_CLEAN_EVENTS
DBA_AUDIT_MGMT_CLEANUP_JOBS
DBA_AUDIT_MGMT_CONFIG_PARAMS
DBA_AUDIT_MGMT_LAST_ARCH_TS
DBA_AUDIT_OBJECT
DBA_AUDIT_POLICIES
DBA_AUDIT_POLICY_COLUMNS
DBA_AUDIT_SESSION
DBA_AUDIT_STATEMENT
DBA_AUDIT_TRAIL
DBA_COMMON_AUDIT_TRAIL
DBA_DV_PATCH_ADMIN_AUDIT
DBA_FGA_AUDIT_TRAIL
DBA_OBJ_AUDIT_OPTS
DBA_OLS_AUDIT_OPTIONS
DBA_PRIV_AUDIT_OPTS
DBA_SA_AUDIT_OPTIONS
DBA_STMT_AUDIT_OPTS
DBA_XS_AUDIT_POLICY_OPTIONS
DBA_XS_AUDIT_TRAIL
DBA_XS_ENABLED_AUDIT_POLICIES

22 filas seleccionadas.
```

Vamos a ver un ejemplo primero con la configuración 'DB' y posteriormente realizaremos el mismo ejemplo para la configuración 'DB,EXTENDED'.

Primero voy a crear un usuario llamado audit_test y vamos a auditar todas las operaciones realizadas por este.

```sql
CREATE USER audit_test IDENTIFIED BY password
  DEFAULT TABLESPACE users
  TEMPORARY TABLESPACE temp
  QUOTA UNLIMITED ON users;

GRANT connect TO audit_test;
GRANT create table, create procedure TO audit_test;

AUDIT ALL BY audit_test BY ACCESS;
AUDIT SELECT TABLE, UPDATE TABLE, INSERT TABLE, DELETE TABLE BY audit_test BY ACCESS;
AUDIT EXECUTE PROCEDURE BY audit_test BY ACCESS;
```

Las opciones anteriores auditan todos los DDL y DML, junto con algunos eventos del sistema.

- DDL (CREATE, ALTER y DROP)
- DML (INSERT, UPDATE, DELETE, SELECT, EXECUTE).
- EVENTOS DEL SISTEMA (LOGON, LOGOFF, etc.)

Nos conectamos con el usuario creado y realizamos algunas operaciones:

```sql
conn audit_test
CREATE TABLE test_tab (
  id  NUMBER
);

INSERT INTO test_tab (id) VALUES (1);
UPDATE test_tab SET id = id;
SELECT * FROM test_tab;
DELETE FROM test_tab;
DROP TABLE test_tab;
```

Comprobamos los registros en la vista dba_audit_trail desde el usuario SYS.

```sql
SELECT username,
       extended_timestamp,
       owner,
       obj_name,
       action_name
FROM   dba_audit_trail
WHERE  owner = 'AUDIT_TEST'
ORDER BY extended_timestamp;
```

**Resultados (DB):**

```sql
USERNAME
--------------------------------------------------------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
OWNER
--------------------------------------------------------------------------------
OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME
----------------------------

AUDIT_TEST
20/02/23 20:10:45,970048 +01:00
AUDIT_TEST
TEST_TAB
CREATE TABLE

AUDIT_TEST
20/02/23 20:10:52,466208 +01:00
AUDIT_TEST
TEST_TAB
INSERT

AUDIT_TEST
20/02/23 20:10:52,473496 +01:00
AUDIT_TEST
TEST_TAB
UPDATE

AUDIT_TEST
20/02/23 20:10:52,477009 +01:00
AUDIT_TEST
TEST_TAB
SELECT

AUDIT_TEST
20/02/23 20:10:52,479997 +01:00
AUDIT_TEST
TEST_TAB
DELETE

AUDIT_TEST
20/02/23 20:10:53,567785 +01:00
AUDIT_TEST
TEST_TAB
DROP TABLE


6 filas seleccionadas.
```

Vamos ahora con el segundo ejemplo (con la configuración 'DB,EXTENDED').

Primero, para cambiar nuestra base de datos del valor 'DB' a 'DB,EXTENDED' utilizamos el siguiente comando:

```sql
ALTER SYSTEM SET audit_trail = DB,EXTENDED SCOPE=SPFILE;
```

Reiniciamos la base de datos.

```sql
shutdown
startup
```

![accesos](/img/audit/3.png)

Comprobamos que ha cambiado:

```sql
show parameter audit;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
audit_file_dest                      string      /opt/oracle/admin/ORCLCDB/adump
audit_syslog_level                   string
audit_sys_operations                 boolean     TRUE
audit_trail                          string      DB, EXTENDED
unified_audit_common_systemlog       string
unified_audit_sga_queue_size         integer     1048576
unified_audit_systemlog              string
```

Seguimos los pasos anteriores. Nos conectamos con el usuario audit_test y realizamos las siguientes operaciones.

```sql
CREATE TABLE test_tab (
  id  NUMBER
);
INSERT INTO test_tab (id) VALUES (1);
UPDATE test_tab SET id = id;
SELECT * FROM test_tab;
DELETE FROM test_tab;
DROP TABLE test_tab;
```

Comprobamos los registros en la vista dba_audit_trail (vemos también las columnas SQL_BIND y SQL_TEXT).

```sql
SELECT username,
       extended_timestamp,
       owner,
       obj_name,
       action_name,
       sql_bind,
       sql_text
FROM   dba_audit_trail
WHERE  owner = 'AUDIT_TEST'
ORDER BY extended_timestamp;
```

**Resultados (DB,EXTENDED):**

```sql
USERNAME
--------------------------------------------------------------------------------
EXTENDED_TIMESTAMP
---------------------------------------------------------------------------
OWNER
--------------------------------------------------------------------------------
OBJ_NAME
--------------------------------------------------------------------------------
ACTION_NAME
----------------------------
SQL_BIND
--------------------------------------------------------------------------------
SQL_TEXT
--------------------------------------------------------------------------------


AUDIT_TEST
20/02/23 20:10:45,970048 +01:00
AUDIT_TEST
TEST_TAB
CREATE TABLE


AUDIT_TEST
20/02/23 20:10:52,466208 +01:00
AUDIT_TEST
TEST_TAB
INSERT


AUDIT_TEST
20/02/23 20:10:52,473496 +01:00
AUDIT_TEST
TEST_TAB
UPDATE


AUDIT_TEST
20/02/23 20:10:52,477009 +01:00
AUDIT_TEST
TEST_TAB
SELECT


AUDIT_TEST
20/02/23 20:10:52,479997 +01:00
AUDIT_TEST
TEST_TAB
DELETE


AUDIT_TEST
20/02/23 20:10:53,567785 +01:00
AUDIT_TEST
TEST_TAB
DROP TABLE


AUDIT_TEST
20/02/23 20:19:51,022584 +01:00
AUDIT_TEST
TEST_TAB
INSERT

INSERT INTO test_tab (id) VALUES (1)


AUDIT_TEST
20/02/23 20:19:51,029225 +01:00
AUDIT_TEST
TEST_TAB
UPDATE

UPDATE test_tab SET id = id


AUDIT_TEST
20/02/23 20:19:51,036064 +01:00
AUDIT_TEST
TEST_TAB
SELECT

SELECT * FROM test_tab


AUDIT_TEST
20/02/23 20:19:51,042390 +01:00
AUDIT_TEST
TEST_TAB
DELETE

DELETE FROM test_tab


AUDIT_TEST
20/02/23 20:19:52,181657 +01:00
AUDIT_TEST
TEST_TAB
DROP TABLE

DROP TABLE test_tab


AUDIT_TEST
20/02/23 20:20:15,615407 +01:00
AUDIT_TEST
TEST_TAB
CREATE TABLE

CREATE TABLE test_tab (
  id  NUMBER
)


AUDIT_TEST
20/02/23 20:20:15,638909 +01:00
AUDIT_TEST
TEST_TAB
INSERT

INSERT INTO test_tab (id) VALUES (1)


AUDIT_TEST
20/02/23 20:20:15,645566 +01:00
AUDIT_TEST
TEST_TAB
UPDATE

UPDATE test_tab SET id = id


AUDIT_TEST
20/02/23 20:20:15,648545 +01:00
AUDIT_TEST
TEST_TAB
SELECT

SELECT * FROM test_tab


AUDIT_TEST
20/02/23 20:20:15,650443 +01:00
AUDIT_TEST
TEST_TAB
DELETE

DELETE FROM test_tab


AUDIT_TEST
20/02/23 20:20:16,421276 +01:00
AUDIT_TEST
TEST_TAB
DROP TABLE

DROP TABLE test_tab

17 filas seleccionadas.
```

Podemos comprobar que en las acciones que se llevaron a cabo con la configuración 'DB' no tienen registros tanto en SQL_BIND como en SQL_TEXT. Sin embargo, en las últimas acciones llevadas a cabo por el usuario audit_test con la configuración 'DB,EXTENDED' sí se han registrado datos en SQL_TEXT.

> [Información interesante sobre Auditoría en Oracle](https://oracle-base.com/articles/10g/auditing-10gr2)

## 7. Averigua si en Postgres se pueden realizar los cuatro primeros apartados. Si es así, documenta el proceso adecuadamente.

Las auditorías como tal no funcionan igual en PostgreSQL a como son en Oracle, pero hay diferentes formas de obtener la información de lo que se pide en los siguientes apartados.

**a) Activar auditoría de intentos de acceso exitosos al sistema.**

Sí se puede obtener información sobre los intentos de acceso exitosos o fallidos a la base de datos. Para ello, se debe configurar el parámetro "log_connections" en el archivo postgresql.conf poniéndolo en 'on'.

Log_connections hace que se registre cada intento de conexión al servidor, así como la finalización exitosa de la autenticación del cliente (si es necesario) y la autorización. Solo los superusuarios y los usuarios con el privilegio SET apropiado pueden cambiar este parámetro al inicio de la sesión y no se puede cambiar en absoluto dentro de una sesión. El valor por defecto es off.

```bash
sudo nano /etc/postgresql/13/main/postgresql.conf

...
log_connections = on
...
```

Reiniciamos postgresql.

```bash
sudo systemctl restart postgresql
```

Cuando se activa este parámetro, PostgreSQL registra en el archivo de registro de eventos (/var/log/postgresql/postgresql-13-main.log) todas las conexiones exitosas al servidor.

Nos conectamos en la base de datos con el usuario postgres.

```bash
sudo su postgres
psql
```

Hacemos también un intento de acceso con un usuario que no existe y comprobamos el archivo de registro log para comprobar los accesos.

```bash
sudo cat /var/log/postgresql/postgresql-13-main.log
```

```txt
...
2023-02-21 13:18:15.563 CET [5540] [desconocido]@[desconocido] LOG:  conexión recibida: host=[local]
2023-02-21 13:18:16.088 CET [5543] [desconocido]@[desconocido] LOG:  conexión recibida: host=[local]
2023-02-21 13:18:16.090 CET [5543] postgres@template1 LOG:  conexión autorizada: usuario=postgres base_de_datos=template1 nombre_de_aplicación=psql
2023-02-21 13:18:16.633 CET [5546] [desconocido]@[desconocido] LOG:  conexión recibida: host=[local]
2023-02-21 13:18:16.636 CET [5546] postgres@template1 LOG:  conexión autorizada: usuario=postgres base_de_datos=template1 nombre_de_aplicación=psql
2023-02-21 13:18:17.162 CET [5549] [desconocido]@[desconocido] LOG:  conexión recibida: host=[local]
2023-02-21 13:18:17.163 CET [5549] postgres@template1 LOG:  conexión autorizada: usuario=postgres base_de_datos=template1 nombre_de_aplicación=psql
2023-02-21 13:18:26.884 CET [5562] [desconocido]@[desconocido] LOG:  conexión recibida: host=[local]
2023-02-21 13:18:26.884 CET [5562] postgres@postgres LOG:  conexión autorizada: usuario=postgres base_de_datos=postgres nombre_de_aplicación=psql
2023-02-21 13:22:42.909 CET [5589] [desconocido]@[desconocido] LOG:  conexión recibida: host=[local]
2023-02-21 13:22:42.910 CET [5589] usuario@usuario FATAL:  la autentificación password falló para el usuario <<usuario>>
2023-02-21 13:22:42.910 CET [5589] usuario@usuario DETALLE:  No existe el rol <<usuario>>.
	La conexión coincidió con la línea 94 de pg_hba.conf: <<local   all             all                         md5>>
2023-02-21 13:22:42.910 CET [5589] usuario@usuario LOG:  could not send data to client: Tubería rota
2023-02-21 13:22:45.110 CET [5591] [desconocido]@[desconocido] LOG:  conexión recibida: host=[local]
2023-02-21 13:22:45.112 CET [5591] usuario@usuario FATAL:  la autentificación password falló para el usuario <<usuario>>
2023-02-21 13:22:45.112 CET [5591] usuario@usuario DETALLE:  No existe el rol <<usuario>>.
	La conexión coincidió con la línea 94 de pg_hba.conf: <<local   all             all                                     md5>>
```

**b) Realizar un procedimiento que muestre los accesos fallidos junto con el motivo de los mismos, transformando el código de error almacenado en un mensaje de texto comprensible.**

He intentado realizar una función o procedimiento que utilizando la función pg_read_file de PostgreSQL lea el archivo de registro '/var/log/postgresql/postgresql-13-main.log' y extraiga la información de los intentos de accesos fallidos, pero no he conseguido hacer que funcione.

**c) Activar la auditoría de las operaciones DML realizadas por un usuario concreto.**

Para poder auditar las operaciones DML realizadas por un usuario se puede activar el parámetro log_statement del fichero de configuración postgresql.conf. Este parámetro admite las opciones:

- none. Valor por defecto en el que no captura ninguna operación.
- ddl. Captura todas las operaciones de definición de datos.
- mod. Lo mismo que ddl pero también registra los inserts, updates y deletes.
- all. Registra todas las sentencias contra la base de datos. Cuando se necesita auditar un sistema es la opción que se suele utilizar.

Con el parámetro log_statement se captura absolutamente todo si lo ponemos a all, el problema es que al recoger demasiada información puede ser complicado filtrarla y encontrar lo que buscamos.

Otra forma de auditar sería creando triggers personalizados. Utilizando las opciones before o after podemos auditar las operaciones en una tabla en concreto. Los triggers permiten personalizar mucho la información que capturamos pero hay que mantenerlos según las estructuras de las tablas que cambian.

Para solventar estos problemas se suele utilizar otras herramientas o extensiones. En mi caso voy a utilizar la extensión Audit-trigger.

Primero lo descargamos.

```bash
wget https://raw.githubusercontent.com/2ndQuadrant/audit-trigger/master/audit.sql
```

Activamos la herramienta en PostgreSQL.

```bash
\i audit.sql
```

Para ver las operaciones DML de un usuario concreto habrá que ir activando esta herramienta tabla por tabla, de la siguiente forma:

```sql
select audit.audit_table('pelicula');
select audit.audit_table('actor');
select audit.audit_table('pelicula_actor');
```

Resultado:

```sql
maravilla=# select audit.audit_table('pelicula');
NOTICE:  disparador \u00abaudit_trigger_row\u00bb para la relaci\u00f3n \u00abpelicula\u00bb no existe, omitiendo
NOTICE:  disparador \u00abaudit_trigger_stm\u00bb para la relaci\u00f3n \u00abpelicula\u00bb no existe, omitiendo
NOTICE:  CREATE TRIGGER audit_trigger_row AFTER INSERT OR UPDATE OR DELETE ON pelicula FOR EACH ROW EXECUTE PROCEDURE audit.if_modified_func('true');
NOTICE:  CREATE TRIGGER audit_trigger_stm AFTER TRUNCATE ON pelicula FOR EACH STATEMENT EXECUTE PROCEDURE audit.if_modified_func('true');
 audit_table 
-------------
 
(1 fila)

maravilla=# select audit.audit_table('actor');
select audit.audit_table('pelicula_actor');
NOTICE:  disparador \u00abaudit_trigger_row\u00bb para la relaci\u00f3n \u00abactor\u00bb no existe, omitiendo
NOTICE:  disparador \u00abaudit_trigger_stm\u00bb para la relaci\u00f3n \u00abactor\u00bb no existe, omitiendo
NOTICE:  CREATE TRIGGER audit_trigger_row AFTER INSERT OR UPDATE OR DELETE ON actor FOR EACH ROW EXECUTE PROCEDURE audit.if_modified_func('true');
NOTICE:  CREATE TRIGGER audit_trigger_stm AFTER TRUNCATE ON actor FOR EACH STATEMENT EXECUTE PROCEDURE audit.if_modified_func('true');
 audit_table 
-------------
 
(1 fila)

NOTICE:  disparador \u00abaudit_trigger_row\u00bb para la relaci\u00f3n \u00abpelicula_actor\u00bb no existe, omitiendo
NOTICE:  disparador \u00abaudit_trigger_stm\u00bb para la relaci\u00f3n \u00abpelicula_actor\u00bb no existe, omitiendo
NOTICE:  CREATE TRIGGER audit_trigger_row AFTER INSERT OR UPDATE OR DELETE ON pelicula_actor FOR EACH ROW EXECUTE PROCEDURE audit.if_modified_func('true');
NOTICE:  CREATE TRIGGER audit_trigger_stm AFTER TRUNCATE ON pelicula_actor FOR EACH STATEMENT EXECUTE PROCEDURE audit.if_modified_func('true');
 audit_table 
-------------
 
(1 fila)
```

Realizamos algunas operaciones con las tablas y sus datos:

```sql
insert into pelicula (codigo,titulo) values ('MA012','Antman Quantumania');
update pelicula set puntuacion = 8 where codigo='MA012';
```

Y comprobamos que se han registrado los movimientos:

```sql
select session_user_name, action, table_name, action_tstamp_clk, client_query from audit.logged_actions;
```

Resultado de la comprobación:

```sql
 session_user_name | action | table_name |       action_tstamp_clk       |                                client_query                                 
-------------------+--------+------------+-------------------------------+-----------------------------------------------------------------------------
 postgres          | I      | pelicula   | 2023-02-21 18:15:29.407968+01 | insert into pelicula (codigo,titulo) values ('MA012','Antman Quantumania');
 postgres          | U      | pelicula   | 2023-02-21 18:16:25.382631+01 | update pelicula set puntuacion = 8 where codigo='MA012';
(2 filas)
```

**d) Realizar una auditoría de grano fino para almacenar información sobre la inserción de empleados con sueldo superior a 2000 en la tabla emp del usuario scott.**

Para hacer esto, la única opción que he encontrado sería utilizar triggers, que permitan la ejecución de código automáticamente en respuesta a ciertos eventos en las tablas de la base de datos. Por ejemplo, crear un trigger para la tabla "emp", para auditar y almacenar información específica sobre la inserción de empleados con sueldo superior a 2000.

## 8. Averigua si en MySQL se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.

**1) Activar auditoría de intentos de acceso exitosos al sistema.**

MySQL no ofrece una opción nativa para auditar los intentos de acceso exitosos al sistema. Sin embargo, se puede utilizar lo registros de errores del servidor MySQL para ver los intentos de acceso fallidos.

Para ver el registro de errores lo hacemos del siguiente modo:

```bash
sudo cat /var/log/mysql/error.log
```

**3) Activar la auditoría de las operaciones DML realizadas por un usuario concreto.**

No existe una opción en MySQL, pero podemos hacerlo con triggers. A continuación veremos un ejemplo.

Primero creamos una base de datos para las auditorías, y creamos una tabla para la salida del trigger.

```sql
create database auditoria;
use auditoria;

CREATE TABLE accesos
 (
   codigo int(11) NOT NULL AUTO_INCREMENT,
   usuario varchar(100),
   fecha datetime,
   PRIMARY KEY (codigo)
 )
 ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;
```

Creamos el trigger.

```sql
delimiter $$
CREATE TRIGGER maravilla.admin
BEFORE INSERT ON maravilla.pelicula
FOR EACH ROW
BEGIN
INSERT INTO auditoria.accesos (usuario, fecha)
values (CURRENT_USER(), NOW());
END$$
```

Entro en mi base de datos llamada maravilla e inserto un registro en la tabla pelicula.

```sql
use maravilla
insert into pelicula (codigo,titulo) values ('MA012','Antman Quantumania');
```

Por último comprobamos que se ha añadido un registro a la tabla accesos de la base de datos audiotria.

```sql
use auditoria
select * from accesos;
```

Resultado:

```sql
+--------+----------------+---------------------+
| codigo | usuario        | fecha               |
+--------+----------------+---------------------+
|      1 | root@localhost | 2023-02-21 18:55:21 |
+--------+----------------+---------------------+
1 row in set (0,00 sec)
```


**4) Realizar una auditoría de grano fino para almacenar información sobre la inserción de empleados con sueldo superior a 2000 en la tabla emp del usuario scott.**

MySQL no ofrece una opción nativa para realizar una auditoría de grano fino como en Oracle.

## 9. Averigua las posibilidades que ofrece MongoDB para auditar los cambios que va sufriendo un documento. Demuestra su funcionamiento.

MongoDB Enterprise incluye la capacidad de auditaría para instancias mongod y mongos. La función de auditoría permite a los administradores y usuarios realizar un seguimiento de la actividad del sistema para implementaciones con múltiples usuarios y aplicaciones.

**Habilitar y configurar la salida de auditoría.**

Para habilitar la auditoría en MongoDB Enterprise, se establece un destino de salida de auditoría con `--auditDestination`.

El destino, donde se escriben los evenos de auditoría pueden ser: en la consola, syslog, un archivo JSON o un archivo BSON.

- **Syslog:**

Para habilitar la auditoría e imprimir eventos de auditoría en syslog en formato JSON, se especifica `syslog` para la configuración `--auditDestination`. Se pueden incluir opciones adicionales según sea necesario para nuestra configuración (por ejemplo la opción `--bind_ip`). La forma en la que se escribiría el comando sería la siguiente:

```bash
mongod --dbpath data/db --auditDestination syslog
```
  
Otra forma  de hacerlo es especificar estas opciones en el archivo de configuración:

```bash
storage:
   dbPath: data/db
auditLog:
   destination: syslog
```

- **Consola:**

Para habilitar la auditoría y mostrar los eventos como salida estándar (es decir, `stdout`), se especifica `console` para la configuración `--auditDestination` (también se pueden especificar algunas opciones adicionales como `--bind_ip`).

```bash
mongod --dbpath data/db --auditDestination console
```

También se puede especificar estas opciones en el archivo de configuración:

```bash
storage:
    dbPath: data/db
auditLog:
    destination: console
```

- **Archivo JSON:**

Para habilitar la auditoría e imprimir eventos de auditoría en un archivo en formato JSON, se especifican las siguientes opciones:

| Opción | Valor |
|------------------------|------------|
| --auditDestination | file |
| --auditFormat | JSON |
| --auditPath | El nombre del archivo de salida. Acepta el nombre de ruta completo o el nombre de ruta relativo. |

Por ejemplo, lo siguiente habilita la auditoría y registra los eventos de auditoría en un archivo con el nombre de ruta relativa data/db/auditLog.json:

```bash
mongod --dbpath data/db --auditDestination file --auditFormat JSON --auditPath data/db/auditLog.json
```

Como en los casos anteriores, también podemos añadir opciones adicionales según sea necesario para la configuración (`--bind_ip`).

El archivo de auditoría se puede rotar con el comando `logRotate`, ya sea junto con el registro del servidor o de forma independiente. Los detalles de la rotación se pueden configurar con la opción de archivo de configuración `systemLog.logRotate` o la opción de línea de comandos `--logRotate`.

También puede especificar estas opciones en el archivo de configuración:

```bash
storage:
   dbPath: data/db
auditLog:
   destination: file
   format: JSON
   path: data/db/auditLog.json
```

La impresión de eventos de auditoría en un archivo en formato JSON degrada el rendimiento del servidor más que la impresión en un archivo en formato BSON.

- **Archivo BSON:**

Para habilitar la auditoría e imprimir eventos en un archivo en formato binario BSON, se especifican las siguientes opciones:

| Opción | Valor |
|------------------------|------------|
| --auditDestination | file |
| --auditFormat | BSON |
| --auditPath | El nombre del archivo de salida. Acepta el nombre de ruta completo o el nombre de ruta relativo. |

Por ejemplo, el siguiente comando habilita la auditoría y registra eventos de auditoría en un archivo BSON con el nombre de ruta relativo de data/db/auditLog.bson:

```bash
mongod --dbpath data/db --auditDestination file --auditFormat BSON --auditPath data/db/auditLog.bson
```

El archivo de auditoría se rota al mismo tiempo que el archivo de registro del servidor. Los detalles de la rotación se pueden configurar con la opción de archivo de configuración `systemLog.logRotate` o la opción de línea de comandos `--logRotate`.

También puede especificar estas opciones en el archivo de configuración:

```bash
storage:
   dbPath: data/db
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
```

Con el siguiente comando se convierte y muestra el registro de auditoría en un formato legible mediante `bsondump`:

```bash
bsondump data/db/auditLog.bson
```

**Qué se alamacena en un registro de auditoría.**

Una vez habilitado, el sistema de auditoría puede registrar las siguientes operaciones:

- Esquema (DDL)
- Conjunto de réplicas y clúster fragmentado
- Autenticación y autorización
- Operaciones CRUD (requiere `auditAuthorizationSuccess` establecido en true).

Además con el sistema de auditoría se pueden configurar filtros para restringir los eventos capturados.

Por último decir que si queremos ver la configuración de auditoría de una instancia mongod o mongos usaremos el siguiente comando:

```bash
db.adminCommand(
   {
     getAuditConfig: 1
   }
)
```

Para más info ver:

- [Comando setAuditConfig](https://www.mongodb.com/docs/manual/reference/command/setAuditConfig/#mongodb-dbcommand-dbcmd.setAuditConfig)
- [Configurar filtros de auditoría](https://www.mongodb.com/docs/manual/tutorial/configure-audit-filters/)
- [Significado de los mensajes de eventos de auditoría](https://www.mongodb.com/docs/manual/reference/audit-message/).

## 10. Averigua si en MongoDB se pueden auditar los accesos a una colección concreta. Demuestra su funcionamiento.

Lo podríamos hacer de la siguiente forma:

```bash
mongod --dbpath data/db --auditDestination file --auditFilter '{
    atype: "authCheck",
    "param.ns": "maravilla.pelicula",
    "param.command": { $in: [ "find", "insert", "delete", "update", "findandmodify" ] }
}'
--auditFormat BSON --auditPath data/db/auditLog.bson
```

En el comando anterior se auditan los accesos a la colección película de la base de datos maravilla cada vez que se utilicen las opciones 'find', 'insert', 'delete', 'update' y 'findandmodify' con los datos de dicha colección.
