---
title: "Movimiento de datos"
date: 2023-03-01T12:18:27+01:00
draft: true
tags: ["bbdd"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

## Movimiento de datos. Exportación e importación

### Creando el esquema Scott

```sql
rlwrap sqlplus / as sysdba
startup
alter session set "_ORACLE_SCRIPT"=true;
```

```sql
SET ECHO OFF

-- Si ya estaba creado el esquema Scott pero lo queremos crear de nuevo borramos el usario con la opción cascade
 
DROP USER SCOTT CASCADE;
 
-- Creamos y damos privilegios a SCOTT, cuya contraseña es TIGER y finalmente nos conectamos con SCOTT/TIGER
 
GRANT CONNECT, RESOURCE, UNLIMITED TABLESPACE TO SCOTT IDENTIFIED BY TIGER;
ALTER USER SCOTT ACCOUNT UNLOCK;
ALTER USER SCOTT DEFAULT TABLESPACE USERS;
ALTER USER SCOTT TEMPORARY TABLESPACE TEMP;
CONN SCOTT/TIGER
 
-- Creación de las tablas de SCOTT
 
CREATE TABLE DUMMY (DUMMY NUMBER);
 
CREATE TABLE DEPT (
        DEPTNO NUMBER(2) CONSTRAINT PK_DEPT PRIMARY KEY,
        DNAME VARCHAR2(14),
        LOC VARCHAR2(13));
 
CREATE TABLE EMP (
        EMPNO NUMBER(4) CONSTRAINT PK_EMP PRIMARY KEY,
        ENAME VARCHAR2(10),
        JOB VARCHAR2(9),
        MGR NUMBER(4),
        HIREDATE DATE,
        SAL NUMBER(7,2),
        COMM NUMBER(7,2),
        DEPTNO NUMBER(2) CONSTRAINT FK_DEPTNO REFERENCES DEPT);
 
CREATE TABLE BONUS (
        ENAME VARCHAR2(10),
        JOB VARCHAR2(9),
        SAL NUMBER,
        COMM NUMBER);
         
CREATE TABLE SALGRADE (
        GRADE NUMBER,
        LOSAL NUMBER,
        HISAL NUMBER);
 
-- Inserción de datos en las tablas
     
INSERT INTO DEPT VALUES (10,'ACCOUNTING','NEW YORK');
INSERT INTO DEPT VALUES (20,'RESEARCH','DALLAS');
INSERT INTO DEPT VALUES (30,'SALES','CHICAGO');
INSERT INTO DEPT VALUES (40,'OPERATIONS','BOSTON');
 
INSERT INTO EMP VALUES (7369,'SMITH','CLERK',7902,to_date('17-12-1980','dd-mm-yyyy'),800,NULL,20);
INSERT INTO EMP VALUES (7499,'ALLEN','SALESMAN',7698,to_date('20-2-1981','dd-mm-yyyy'),1600,300,30);
INSERT INTO EMP VALUES (7521,'WARD','SALESMAN',7698,to_date('22-2-1981','dd-mm-yyyy'),1250,500,30);
INSERT INTO EMP VALUES (7566,'JONES','MANAGER',7839,to_date('2-4-1981','dd-mm-yyyy'),2975,NULL,20);
INSERT INTO EMP VALUES (7654,'MARTIN','SALESMAN',7698,to_date('28-9-1981','dd-mm-yyyy'),1250,1400,30);
INSERT INTO EMP VALUES (7698,'BLAKE','MANAGER',7839,to_date('1-5-1981','dd-mm-yyyy'),2850,NULL,30);
INSERT INTO EMP VALUES (7782,'CLARK','MANAGER',7839,to_date('9-6-1981','dd-mm-yyyy'),2450,NULL,10);
INSERT INTO EMP VALUES (7788,'SCOTT','ANALYST',7566,to_date('13-JUL-87')-85,3000,NULL,20);
INSERT INTO EMP VALUES (7839,'KING','PRESIDENT',NULL,to_date('17-11-1981','dd-mm-yyyy'),5000,NULL,10);
INSERT INTO EMP VALUES (7844,'TURNER','SALESMAN',7698,to_date('8-9-1981','dd-mm-yyyy'),1500,0,30);
INSERT INTO EMP VALUES (7876,'ADAMS','CLERK',7788,to_date('13-JUL-87')-51,1100,NULL,20);
INSERT INTO EMP VALUES (7900,'JAMES','CLERK',7698,to_date('3-12-1981','dd-mm-yyyy'),950,NULL,30);
INSERT INTO EMP VALUES (7902,'FORD','ANALYST',7566,to_date('3-12-1981','dd-mm-yyyy'),3000,NULL,20);
INSERT INTO EMP VALUES (7934,'MILLER','CLERK',7782,to_date('23-1-1982','dd-mm-yyyy'),1300,NULL,10);
 
INSERT INTO SALGRADE VALUES (1,700,1200);
INSERT INTO SALGRADE VALUES (2,1201,1400);
INSERT INTO SALGRADE VALUES (3,1401,2000);
INSERT INTO SALGRADE VALUES (4,2001,3000);
INSERT INTO SALGRADE VALUES (5,3001,9999);

INSERT INTO DUMMY VALUES (0);
 
COMMIT;
 
SET TERMOUT ON
SET ECHO ON
```

### 1. Realiza una exportación del esquema de SCOTT usando Oracle Data Pump con las siguientes condiciones:

**• Exporta tanto la estructura de las tablas como los datos de las mismas.**
**• Excluye la tabla BONUS y los departamentos con menos de dos empleados.**
**• Realiza una estimación previa del tamaño necesario para el fichero de exportación.**
**• Programa la operación para dentro de 2 minutos.**
**• Genera un archivo de log en el directorio raíz.**

Le damos permisos a Scott para exportar datos.

```sql
GRANT DATAPUMP_EXP_FULL_DATABASE TO SCOTT;
```

Para realizar la exportación del esquema SCOTT utilizando Oracle Data Pump con las condiciones especificadas, podemos utilizar el siguiente comando desde la terminal de Debian:

```bash
echo "expdp SCOTT/TIGER DIRECTORY=DATA_PUMP_DIR DUMPFILE=scott.dmp LOGFILE=scott.log ESTIMATE=STATISTICS EXCLUDE=TABLE:\"=\'BONUS\'\"  QUERY=dept:'"WHERE deptno IN \(SELECT deptno FROM EMP GROUP BY deptno HAVING COUNT\(*\)>2\)"' SCHEMAS=scott COMPRESSION=ALL" | at now + 2 minutes
```

Explicación del comando:

- DIRECTORY: indica el directorio en el que se va a generar el archivo de exportación. En este caso, se utiliza el directorio predefinido DATA_PUMP_DIR de Oracle.
- DUMPFILE: especifica el nombre del archivo de exportación. En este caso, se utiliza el nombre scott.dmp.
- LOGFILE: indica el nombre del archivo de log. En este caso, se utiliza el nombre scott.log.
- ESTIMATE: genera una estimación previa del tamaño necesario para el fichero de exportación.
- EXCLUDE: excluye la tabla BONUS
- QUERY: exporta solo los datos de la consulta, en este caso, de la tabla dept exporta todos los departamentos con más de dos empleados, o lo que es lo mismo, exporta todos los datos exceptuando los departamentos con menos de dos empleados.
- SCHEMAS: indica el esquema que se va a exportar. En este caso, es el esquema SCOTT.
- COMPRESSION: indica el nivel de compresión que se va a utilizar para el archivo de exportación. En este caso, se utiliza el nivel máximo de compresión (ALL).
- at now + 2 minutes: indica que el comando se ejecutará dentro de 2 minutos.

Una vez ejecutado el comando, se generará un archivo de exportación en el directorio DATA_PUMP_DIR (/opt/oracle/admin/ORCLCDB/dpdump/) con el nombre scott.dmp y un archivo de log con el nombre scott.log.

**Resultado:**

```bash
Export: Release 19.0.0.0.0 - Production on Wed Mar 1 14:34:54 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "SCOTT"."SYS_EXPORT_SCHEMA_01":  SCOTT/******** DIRECTORY=DATA_PUMP_DIR DUMPFILE=scott.dmp LOGFILE=scott.log ESTIMATE=STATISTICS EXCLUDE=TABLE:"='BONUS'" QUERY=dept:"WHERE deptno IN \(SELECT deptno FROM EMP GROUP BY deptno HAVING COUNT\(*\)>2\)" SCHEMAS=scott COMPRESSION=ALL 
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
.  estimado "SCOTT"."DEPT"                              4.683 KB
.  estimado "SCOTT"."DUMMY"                             4.683 KB
.  estimado "SCOTT"."EMP"                               4.683 KB
.  estimado "SCOTT"."SALGRADE"                          4.683 KB
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
Procesando el tipo de objeto SCHEMA_EXPORT/ROLE_GRANT
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/COMMENT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/INDEX
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
. . "SCOTT"."DEPT"                              4.953 KB       3 filas exportadas
. . "SCOTT"."DUMMY"                             4.687 KB       1 filas exportadas
. . "SCOTT"."EMP"                               5.679 KB      14 filas exportadas
. . "SCOTT"."SALGRADE"                          4.906 KB       5 filas exportadas
La tabla maestra "SCOTT"."SYS_EXPORT_SCHEMA_01" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para SCOTT.SYS_EXPORT_SCHEMA_01 es:
  /opt/oracle/admin/ORCLCDB/dpdump/scott.dmp
El trabajo "SCOTT"."SYS_EXPORT_SCHEMA_01" ha terminado correctamente en Mie Mar 1 14:35:34 2023 elapsed 0 00:00:40
```

![exportacion](/img/movdatos/1.png)


### 2. Importa el fichero obtenido anteriormente usando Oracle Data Pump pero en un usuario distinto de la misma base de datos.

Concedemos el permiso para realizar la importación al usuario llamado 'ara', en el caso de que no lo tuviera.

```sql
GRANT IMP_FULL_DATABASE TO ARA;
```

Importamos los datos para el usuario 'ara'.

```bash
impdp ara/ara schemas=scott directory=DATA_PUMP_DIR dumpfile=scott.dmp logfile=scott.log REMAP_SCHEMA=SCOTT:ara
```

**Resultado:**

```bash
Import: Release 19.0.0.0.0 - Production on Thu Mar 2 13:07:33 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

La tabla maestra "ARA"."SYS_IMPORT_SCHEMA_01" se ha cargado/descargado correctamente
Iniciando "ARA"."SYS_IMPORT_SCHEMA_01":  ara/******** schemas=scott directory=DATA_PUMP_DIR dumpfile=scott.dmp logfile=scott.log REMAP_SCHEMA=SCOTT:ara 
Procesando el tipo de objeto SCHEMA_EXPORT/ROLE_GRANT
Procesando el tipo de objeto SCHEMA_EXPORT/DEFAULT_ROLE
Procesando el tipo de objeto SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/TABLE_DATA
. . "ARA"."DEPT"                                4.953 KB       3 filas importadas
. . "ARA"."DUMMY"                               4.687 KB       1 filas importadas
. . "ARA"."EMP"                                 5.679 KB      14 filas importadas
. . "ARA"."SALGRADE"                            4.906 KB       5 filas importadas
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Procesando el tipo de objeto SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto SCHEMA_EXPORT/STATISTICS/MARKER
El trabajo "ARA"."SYS_IMPORT_SCHEMA_01" ha terminado correctamente en Jue Mar 2 13:07:37 2023 elapsed 0 00:00:04
```

![exportacion](/img/movdatos/2.png)

### 3. Realiza una exportación de la estructura de todas las tablas de la base de datos usando el comando expdp de Oracle Data Pump probando al menos cinco de las posibles opciones que ofrece dicho comando y documentándolas adecuadamente.

En el ejercicio 1 ya expliqué detalladamente lo que hacía cada parámetro del comando expdp de Oracle Data Pump, pero ahora vamos a hacerlo con todas las tablas de la base de datos de forma más sencilla.

```bash
expdp system/system FULL=y DIRECTORY=DATA_PUMP_DIR DUMPFILE=full_db.dmp EXCLUDE=INDEX JOB_NAME=export_full
```

- FULL: realiza una exportación completa de la base de datos incluyendo todas las tablas, índices, procedimientos almacenados, etc.

- DIRECTORY: especifica el directorio de sistema de archivos donde se guardarán los archivos de exportación en este caso usará el directorio por defecto de Oracle para guardar los archivos de exportación (/opt/oracle/admin/ORCLCDB/dpdump/).

- DUMPFILE: especifica el nombre del archivo de volcado que se creará en el directorio especificado, en este caso el nombre del archivo es full_db.dmp.

- EXCLUDE: especifica los objetos que se excluirán de la exportación. En este caso, podemos excluir los índices de todas las tablas usando el parámetro EXCLUDE=INDEX.

- JOB_NAME: asigna un nombre al trabajo de exportación, en este caso export_full.

**Resultado:**

```bash
Export: Release 19.0.0.0.0 - Production on Thu Mar 2 13:36:14 2023
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production

Advertencia: Las operaciones de Oracle Data Pump no se necesitan normalmente cuando se conecta a la raiz o al elemento inicial de una base de datos del contenedor.

Iniciando "SYSTEM"."EXPORT_FULL":  system/******** FULL=y DIRECTORY=DATA_PUMP_DIR DUMPFILE=full_db.dmp EXCLUDE=INDEX JOB_NAME=export_full 
Procesando el tipo de objeto DATABASE_EXPORT/EARLY_OPTIONS/VIEWS_AS_TABLES/TABLE_DATA
Procesando el tipo de objeto DATABASE_EXPORT/NORMAL_OPTIONS/TABLE_DATA
Procesando el tipo de objeto DATABASE_EXPORT/NORMAL_OPTIONS/VIEWS_AS_TABLES/TABLE_DATA
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/TABLE_DATA
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/STATISTICS/TABLE_STATISTICS
Procesando el tipo de objeto DATABASE_EXPORT/STATISTICS/MARKER
Procesando el tipo de objeto DATABASE_EXPORT/PRE_SYSTEM_IMPCALLOUT/MARKER
Procesando el tipo de objeto DATABASE_EXPORT/PRE_INSTANCE_IMPCALLOUT/MARKER
Procesando el tipo de objeto DATABASE_EXPORT/TABLESPACE
Procesando el tipo de objeto DATABASE_EXPORT/PROFILE
Procesando el tipo de objeto DATABASE_EXPORT/RADM_FPTM
Procesando el tipo de objeto DATABASE_EXPORT/GRANT/SYSTEM_GRANT/PROC_SYSTEM_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/GRANT/SYSTEM_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/ROLE_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/DEFAULT_ROLE
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/ON_USER_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLESPACE_QUOTA
Procesando el tipo de objeto DATABASE_EXPORT/RESOURCE_COST
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/DB_LINK
Procesando el tipo de objeto DATABASE_EXPORT/TRUSTED_DB_LINK
Procesando el tipo de objeto DATABASE_EXPORT/DIRECTORY/DIRECTORY
Procesando el tipo de objeto DATABASE_EXPORT/DIRECTORY/GRANT/OWNER_GRANT/OBJECT_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SYSTEM_PROCOBJACT/PRE_SYSTEM_ACTIONS/PROCACT_SYSTEM
Procesando el tipo de objeto DATABASE_EXPORT/SYSTEM_PROCOBJACT/PROCOBJ
Procesando el tipo de objeto DATABASE_EXPORT/SYSTEM_PROCOBJACT/POST_SYSTEM_ACTIONS/PROCACT_SYSTEM
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto DATABASE_EXPORT/EARLY_OPTIONS/VIEWS_AS_TABLES/TABLE
Procesando el tipo de objeto DATABASE_EXPORT/EARLY_POST_INSTANCE_IMPCALLOUT/MARKER
Procesando el tipo de objeto DATABASE_EXPORT/NORMAL_OPTIONS/TABLE
Procesando el tipo de objeto DATABASE_EXPORT/NORMAL_OPTIONS/VIEWS_AS_TABLES/TABLE
Procesando el tipo de objeto DATABASE_EXPORT/NORMAL_POST_INSTANCE_IMPCALLOUT/MARKER
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/TABLE
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/COMMENT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/PACKAGE/PACKAGE_SPEC
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/PACKAGE/CODE_BASE_GRANT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/FUNCTION/FUNCTION
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/PROCEDURE/PROCEDURE
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/PACKAGE/COMPILE_PACKAGE/PACKAGE_SPEC/ALTER_PACKAGE_SPEC
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/FUNCTION/ALTER_FUNCTION
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/PROCEDURE/ALTER_PROCEDURE
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/CONSTRAINT/CONSTRAINT
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/TABLE/CONSTRAINT/REF_CONSTRAINT
Procesando el tipo de objeto DATABASE_EXPORT/FINAL_POST_INSTANCE_IMPCALLOUT/MARKER
Procesando el tipo de objeto DATABASE_EXPORT/SCHEMA/POST_SCHEMA/PROCACT_SCHEMA
Procesando el tipo de objeto DATABASE_EXPORT/AUDIT_UNIFIED/AUDIT_POLICY_ENABLE
Procesando el tipo de objeto DATABASE_EXPORT/AUDIT
Procesando el tipo de objeto DATABASE_EXPORT/POST_SYSTEM_IMPCALLOUT/MARKER
. . "SYS"."KU$_USER_MAPPING_VIEW"               6.195 KB      45 filas exportadas
. . "AUDSYS"."AUD$UNIFIED":"SYS_P181"           131.0 KB     159 filas exportadas
. . "AUDSYS"."AUD$UNIFIED":"SYS_P454"           64.37 KB      15 filas exportadas
. . "AUDSYS"."AUD$UNIFIED":"SYS_P330"           57.76 KB      15 filas exportadas
. . "AUDSYS"."AUD$UNIFIED":"SYS_P650"           91.28 KB     104 filas exportadas
. . "SYSTEM"."REDO_DB"                          25.59 KB       1 filas exportadas
. . "WMSYS"."WM$WORKSPACES_TABLE$"              12.10 KB       1 filas exportadas
. . "WMSYS"."WM$HINT_TABLE$"                    9.984 KB      97 filas exportadas
. . "LBACSYS"."OLS$INSTALLATIONS"               6.960 KB       2 filas exportadas
. . "WMSYS"."WM$WORKSPACE_PRIV_TABLE$"          7.078 KB      11 filas exportadas
. . "SYS"."DAM_CONFIG_PARAM$"                   6.531 KB      14 filas exportadas
. . "SYS"."TSDP_SUBPOL$"                        6.328 KB       1 filas exportadas
. . "WMSYS"."WM$NEXTVER_TABLE$"                 6.375 KB       1 filas exportadas
. . "LBACSYS"."OLS$PROPS"                       6.234 KB       5 filas exportadas
. . "WMSYS"."WM$ENV_VARS$"                      6.015 KB       3 filas exportadas
. . "SYS"."TSDP_PARAMETER$"                     5.953 KB       1 filas exportadas
. . "SYS"."TSDP_POLICY$"                        5.921 KB       1 filas exportadas
. . "WMSYS"."WM$VERSION_HIERARCHY_TABLE$"       5.984 KB       1 filas exportadas
. . "WMSYS"."WM$EVENTS_INFO$"                   5.812 KB      12 filas exportadas
. . "LBACSYS"."OLS$AUDIT_ACTIONS"               5.757 KB       8 filas exportadas
. . "LBACSYS"."OLS$DIP_EVENTS"                  5.539 KB       2 filas exportadas
. . "AUDSYS"."AUD$UNIFIED":"AUD_UNIFIED_P0"         0 KB       0 filas exportadas
. . "AUDSYS"."AUD$UNIFIED":"SYS_P790"           148.7 KB      67 filas exportadas
. . "LBACSYS"."OLS$AUDIT"                           0 KB       0 filas exportadas
. . "LBACSYS"."OLS$COMPARTMENTS"                    0 KB       0 filas exportadas
. . "LBACSYS"."OLS$DIP_DEBUG"                       0 KB       0 filas exportadas
. . "LBACSYS"."OLS$GROUPS"                          0 KB       0 filas exportadas
. . "LBACSYS"."OLS$LAB"                             0 KB       0 filas exportadas
. . "LBACSYS"."OLS$LEVELS"                          0 KB       0 filas exportadas
. . "LBACSYS"."OLS$POL"                             0 KB       0 filas exportadas
. . "LBACSYS"."OLS$POLICY_ADMIN"                    0 KB       0 filas exportadas
. . "LBACSYS"."OLS$POLS"                            0 KB       0 filas exportadas
. . "LBACSYS"."OLS$POLT"                            0 KB       0 filas exportadas
. . "LBACSYS"."OLS$PROFILE"                         0 KB       0 filas exportadas
. . "LBACSYS"."OLS$PROFILES"                        0 KB       0 filas exportadas
. . "LBACSYS"."OLS$PROG"                            0 KB       0 filas exportadas
. . "LBACSYS"."OLS$SESSINFO"                        0 KB       0 filas exportadas
. . "LBACSYS"."OLS$USER"                            0 KB       0 filas exportadas
. . "LBACSYS"."OLS$USER_COMPARTMENTS"               0 KB       0 filas exportadas
. . "LBACSYS"."OLS$USER_GROUPS"                     0 KB       0 filas exportadas
. . "LBACSYS"."OLS$USER_LEVELS"                     0 KB       0 filas exportadas
. . "SYS"."AUD$"                                95.07 KB     187 filas exportadas
. . "SYS"."DAM_CLEANUP_EVENTS$"                     0 KB       0 filas exportadas
. . "SYS"."DAM_CLEANUP_JOBS$"                       0 KB       0 filas exportadas
. . "SYS"."TSDP_ASSOCIATION$"                       0 KB       0 filas exportadas
. . "SYS"."TSDP_CONDITION$"                         0 KB       0 filas exportadas
. . "SYS"."TSDP_FEATURE_POLICY$"                    0 KB       0 filas exportadas
. . "SYS"."TSDP_PROTECTION$"                        0 KB       0 filas exportadas
. . "SYS"."TSDP_SENSITIVE_DATA$"                    0 KB       0 filas exportadas
. . "SYS"."TSDP_SENSITIVE_TYPE$"                    0 KB       0 filas exportadas
. . "SYS"."TSDP_SOURCE$"                            0 KB       0 filas exportadas
. . "SYSTEM"."REDO_LOG"                             0 KB       0 filas exportadas
. . "WMSYS"."WM$BATCH_COMPRESSIBLE_TABLES$"         0 KB       0 filas exportadas
. . "WMSYS"."WM$CONS_COLUMNS$"                      0 KB       0 filas exportadas
. . "WMSYS"."WM$CONSTRAINTS_TABLE$"                 0 KB       0 filas exportadas
. . "WMSYS"."WM$LOCKROWS_INFO$"                     0 KB       0 filas exportadas
. . "WMSYS"."WM$MODIFIED_TABLES$"                   0 KB       0 filas exportadas
. . "WMSYS"."WM$MP_GRAPH_WORKSPACES_TABLE$"         0 KB       0 filas exportadas
. . "WMSYS"."WM$MP_PARENT_WORKSPACES_TABLE$"        0 KB       0 filas exportadas
. . "WMSYS"."WM$NESTED_COLUMNS_TABLE$"              0 KB       0 filas exportadas
. . "WMSYS"."WM$RESOLVE_WORKSPACES_TABLE$"          0 KB       0 filas exportadas
. . "WMSYS"."WM$RIC_LOCKING_TABLE$"                 0 KB       0 filas exportadas
. . "WMSYS"."WM$RIC_TABLE$"                         0 KB       0 filas exportadas
. . "WMSYS"."WM$RIC_TRIGGERS_TABLE$"                0 KB       0 filas exportadas
. . "WMSYS"."WM$UDTRIG_DISPATCH_PROCS$"             0 KB       0 filas exportadas
. . "WMSYS"."WM$UDTRIG_INFO$"                       0 KB       0 filas exportadas
. . "WMSYS"."WM$VERSION_TABLE$"                     0 KB       0 filas exportadas
. . "WMSYS"."WM$VT_ERRORS_TABLE$"                   0 KB       0 filas exportadas
. . "WMSYS"."WM$WORKSPACE_SAVEPOINTS_TABLE$"        0 KB       0 filas exportadas
. . "MDSYS"."RDF_PARAM$"                        6.515 KB       3 filas exportadas
. . "SYS"."AUDTAB$TBS$FOR_EXPORT"               5.953 KB       2 filas exportadas
. . "SYS"."DBA_SENSITIVE_DATA"                      0 KB       0 filas exportadas
. . "SYS"."DBA_TSDP_POLICY_PROTECTION"              0 KB       0 filas exportadas
. . "SYS"."FGA_LOG$FOR_EXPORT"                  18.07 KB       2 filas exportadas
. . "SYS"."NACL$_ACE_EXP"                           0 KB       0 filas exportadas
. . "SYS"."NACL$_HOST_EXP"                      7.046 KB       3 filas exportadas
. . "SYS"."NACL$_WALLET_EXP"                        0 KB       0 filas exportadas
. . "SYS"."SQL$_DATAPUMP"                           0 KB       0 filas exportadas
. . "SYS"."SQLOBJ$AUXDATA_DATAPUMP"                 0 KB       0 filas exportadas
. . "SYS"."SQLOBJ$DATA_DATAPUMP"                    0 KB       0 filas exportadas
. . "SYS"."SQLOBJ$_DATAPUMP"                        0 KB       0 filas exportadas
. . "SYS"."SQLOBJ$PLAN_DATAPUMP"                    0 KB       0 filas exportadas
. . "SYS"."SQL$TEXT_DATAPUMP"                       0 KB       0 filas exportadas
. . "SYSTEM"."SCHEDULER_JOB_ARGS"                   0 KB       0 filas exportadas
. . "SYSTEM"."SCHEDULER_PROGRAM_ARGS"               0 KB       0 filas exportadas
. . "WMSYS"."WM$EXP_MAP"                        7.718 KB       3 filas exportadas
. . "WMSYS"."WM$METADATA_MAP"                       0 KB       0 filas exportadas
. . "RAUL"."PROFESORES"                         5.539 KB       2 filas exportadas
. . "ADMIN"."ACTOR"                             7.148 KB      28 filas exportadas
. . "ADMIN"."PELICULA"                          7.218 KB      10 filas exportadas
. . "ADMIN"."PELICULA_ACTOR"                    7.703 KB      75 filas exportadas
. . "ARA"."DEPT"                                    6 KB       3 filas exportadas
. . "ARA"."DUMMY"                               5.054 KB       1 filas exportadas
. . "ARA"."EMP"                                 8.773 KB      14 filas exportadas
. . "ARA"."SALGRADE"                            5.953 KB       5 filas exportadas
. . "SCOTT"."BONUS"                                 0 KB       0 filas exportadas
. . "SCOTT"."DEPT"                                  6 KB       3 filas exportadas
. . "SCOTT"."DUMMY"                             5.054 KB       1 filas exportadas
. . "SCOTT"."EMP"                               8.773 KB      14 filas exportadas
. . "SCOTT"."SALGRADE"                          5.953 KB       5 filas exportadas
La tabla maestra "SYSTEM"."EXPORT_FULL" se ha cargado/descargado correctamente
******************************************************************************
El juego de archivos de volcado para SYSTEM.EXPORT_FULL es:
  /opt/oracle/admin/ORCLCDB/dpdump/full_db.dmp
El trabajo "SYSTEM"."EXPORT_FULL" ha terminado correctamente en Jue Mar 2 13:38:47 2023 elapsed 0 00:02:33
```

### 4. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con MySQL desde línea de comandos, documentando el proceso.

### 5. Intenta realizar operaciones similares de importación y exportación con las herramientas proporcionadas con Postgres desde línea de comandos, documentando el proceso.

### 6. Exporta los documentos de una colección de MongoDB que cumplan una determinada condición e impórtalos en otra base de datos.

### 7. SQL*Loader es una herramienta que sirve para cargar grandes volúmenes de datos en una instancia de ORACLE.

### Exportad los datos de una base de datos completa desde Postgres a texto plano con delimitadores y emplead SQL*Loader para realizar el proceso de carga de dichos datos a una instancia ORACLE.

### Debéis documentar todo el proceso, explicando los distintos ficheros de configuración y de log que tiene SQL*Loader.
