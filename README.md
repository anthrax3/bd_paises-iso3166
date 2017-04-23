Existen diferentes estándares con los cuales se identifican los países,  para este ejercicio utilizaremos el estándar ISO 3166 - https://www.iso.org/iso-3166-country-codes.html.

 
ISO 3166

La ISO 3166 es un estándar internacional para los códigos de país y los códigos para sus subdivisiones. Ha sido publicada por la Organización Internacional de Normalización. El propósito de la norma ISO 3166 es el establecimiento de códigos reconocidos de manera internacional para la representación de nombres de países, territorios o áreas de interés geográficos la lista de valores se encuentra para consulta en - https://www.iso.org/iso-3166-country-codes.html

 
Ecenario

Se cuenta con acceso remoto a una base de datos en un servidor con PostgreSQL y se requiere que se cree la tabla de países de acuerdo al diccionario de datos siguiente, los entregables son: la sentencia o consulta utilizada para la creación del archivo CSV final, utilizando las tablas creadas de archivos CSV, la sentencia de creación de la tabla de acuerdo al diccionario de datos y el respaldo de la tabla utilizando la herramienta dump.

 
Diccionario de Datos
# 	nombre 	tipo 	valor 	Descripción
1 	iso3166_1 	char 	3 	Código numérico - ISO 3166-1
2 	iso3166_2 	char 	2 	Código de dos letras - ISO 3166-2
3 	iso3166_3 	char 	3 	Código de tres letras - ISO 3166-3
4 	nombre_espanol 	varchar 	 60 	Nombres de los países en lenguaje español
5 	nombre_ingles 	varchar 	 60 	Nombres de los países en lenguaje inglés
6 	nombre_chino 	varchar 	 60 	Nombres de los países en lenguaje chino
7 	nombre_ruso 	varchar 	 60 	Nombres de los países en lenguaje ruso
8 	nombre_frances 	varchar 	 60 	Nombres de los países en lenguaje francés
9 	nombre_arabe 	varchar 	 60 	Nombres de los países en lenguaje árabe
10 	latitud 	float8 	  	Latitud en decimal
11 	longitud 	float8 	  	Longitud en decimal
12 	m49 	char 	3 	Número de país asignado por la ONU
13 	estatus_desarrollo 	varchar 	10 	Pais desarrollado o en desarrollo
14 	codigo_region 	char 	3 	Codigo ONU para la región
15 	codigo_sub_region 	char 	3 	Codigo ONU para la sub región
16 	nombre_region_espanol 	varchar 	30 	Nombre de la región en español
17 	nombre_region_ingles 	varchar 	30 	Nombre de la región en ingles
18 	nombre_region_chino 	varchar 	30 	Nombre de la región en chino
19 	nombre_region_ruso 	varchar 	30 	Nombre de la región en ruso
20 	nombre_region_frances 	varchar 	30 	Nombre de la región en francés
21 	nombre_region_arabe 	varchar 	30 	Nombre de la región en árabe
22 	nombre_subregion_espanol 	varchar 	40 	Nombre de la sub región en español
23 	nombre_subregion_ingles 	varchar 	40 	Nombre de la sub región en inglés
24 	nombre_subregion_chino 	varchar 	40 	Nombre de la sub región en chino
25 	nombre_subregion_ruso 	varchar 	40 	Nombre de la sub región en ruso
26 	nombre_subregion_frances 	varchar 	40 	Nombre de la sub región en francés
27 	nombre_subregion_arabe 	varchar 	40 	Nombre de la sub región en árabe
28 	itu 	char 	3 	Códigos asignados por la International Telecommunications Union
29 	wmo 	char 	2 	Códigos asignados por la World Meteorological Organization
30 	dial 	varchar 	20 	Código del país para marcado telefónico de la recomendación ITU-T - 164
31 	fifa 	varchar 	20 	Códigos asignados por la Fédération Internationale de Football Association
32 	ioc 	char 	3 	Códigos asignados por el International Olympics Committee
33 	tld 	char 	3 	Dominios de Internet
34 	lenguajes 	varchar 	100 	Lenguajes que se hablan en el país
35 	codigo_alfabetico_moneda 	char 	3 	Código alfabético de la moneda
36 	nombre_moneda_ingles 	varchar 	50 	Nombre de la moneda
37 	codigo_numerico_moneda 	smallint 	  	Código numérico asignado a la moneda
38 	edgar 	char 	2 	Códigos asignados por sec.gov

 
Obtención de la información en formato CSV

Lo primero es obtener el listado completo de acuerdo al estándar ISO 3166, como tiene costo el tener acceso a los archivos de descarga, ya sea que creemos el archivo en base a los datos mostrados en el sitio oficial de ISO, o como en este caso obtendremos el listado desde github, https://github.com/openmundi/world.csv/blob/master/attic/countries(249)_iso.csv, este archivo nos sirve como base ya que contiene cuatro de los campos requeridos en el ejercicio: iso3166-1, iso3166-2, iso3166-3 y nombre_ingles, sin embargo el nombre oficial tanto en inglés como en otros idiomas los obtendremos desde otra fuente de información. Archivos - 1.

La latitud y longitud aproximada de los países descargando la hoja de cálculo en formato CSV publicado en https://www.ohadpr.com/2010/04/countries-approximate-lat-lon-and-iso-3166-1-alpha-2/. Archivos - 1.

El código de la ONU "M49" así como los nombres de los países, regiones y subregiones en español, inglés, chino, ruso, francés y árabe, así como el dato de si se encuentra o no en desarrollo, información que se obtiene con los datos publicados por la en https://unstats.un.org/unsd/methodology/m49/overview/ (más datos de la ONU disponibles en - http://data.un.org/Default.aspx). Archivos - 6.

El resto de la información la obtienes de Frictionless Data en http://data.okfn.org/data/core/country-codes#data. Archivos - 1.

 
Creación de Tablas

Una vez que se tienen los archivos con la información en formato CSV, será necesario que se importen los archivos a tablas en la base de datos.

Para crear la tabla en PostgreSQL será necesario conocer el orden de los datos en nuestro archivo,  es decir el orden de las columnas que tiene nuestro archivo CSV, esto se puede hacer viendo la primer fila del archivo, esto lo hacemos con el editor de texto que tengamos en nuestro sistema o en el caso de sistemas tipo Unix como Linux, con el comando head .

# head -n 5 archivo.csv

 

Para el tipo de dato debemos seguir la documentación de PostgreSQL en https://www.postgresql.org/docs/9.2/static/datatype.html, para  crear la tabla con el tipo de dato correcto, el tamaño del campo para los tipos de datos cadenas (strings) como char o varchar, pondremos un valor lo suficientemente alto para que se puedan ingresar los datos a la tabla producto de la importación del archivo CSV, para después mediante las funciones max() y length(), encontremos el tamaño de la cadena más grande en una columna.

=# SELECT MAX(LENGTH(columna)) FROM tabla;

 

 Una de las ventajas de los sistemas de bases de datos como PostgreSQL, es que podemos anidar las consultas, es decir que podemos realizar una consulta o query del resultado de otra consulta.

SELECT tabla_1.campo_1, tabla_2.campo_1 FROM (SELECT * FROM tabla_1 ) LEFT JOIN tabla_2 ON tabla_1.campo_1 = tabla_2.campo_1 ORDER BY tabla_1.campo_1 DESC;

 

Utilizamos "\copy" en lugar de "COPY" ya que de la primera forma hace referencia a la terminal local desde la que estamos conectados al servidor de PostgreSQL, cuando utilizamos "COPY" PostgreSQL hace referencia al servidor como equipo local - https://www.postgresql.org/docs/9.2/static/sql-copy.html.

 

 
Tabla ISO 3166

De acuerdo con el encabezado del archivo CSV que contiene los códigos ISO3166, se encuentran los campos: Name,Alpha-2,Alpha-3 y Numeric-3, por lo que para crear la tabla en PostgreSQL sería la siguiente sentencia.

CREATE TABLE csv_iso3166 (
   nombre_pais   varchar(50),
   iso3166_2     char(2),
   iso3166_3     char(3),
   iso3166_1     char(3)
);

Importamos el contenido del archivo countries_249_iso.csv

# \copy csv_iso3166 FROM '/ruta_al_archivo/.../countries_249_iso.csv' DELIMITER ',' CSV HEADER;

 
Tabla Latitud Longitud

Obtenemos el encabezado del archivo "country_lat_long.csv" para la creación de la tabla en la base de datos en PostgreSQL.

CREATE TABLE csv_latitud_longitud (
   nombre_pais   varchar(50),
   iso3166_2     char(2),
   latitud       float8,
   longitud      float8
); 

Importamos el contenido del archivo countries_249_iso.csv

# \copy csv_latitud_longitud FROM '/ruta_al_archivo/.../country_lat_long.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 
Tabla ONU Español

Obtenemos el encabezado del archivo "onu_espanol.csv" para la creación de la tabla de datos.

CREATE TABLE csv_onu_espanol(
   global_code                char(3),
   global_name                varchar(250),
   codigo_region              char(3),
   nombre_region_espanol      varchar(30),
   codigo_sub_region          char(3),
   nombre_subregion_espanol   varchar(40),
   intermediate_region_code   varchar(250),
   intermediate_region_name   varchar(250),
   nombre_espanol             varchar(60),
   m49                        char(3),
   iso3166_3                  char(3),
   ldc                        char(1),
   lldc                       char(1),
   sids                       char(1),
   estatus_desarrollo         varchar(15)
); 

Importamos el contenido del archivo onu_espanol.csv

# \copy csv_onu_espanol FROM '/ruta_al_archivo/.../onu_espanol.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 
Tabla ONU Inglés

Obtenemos el encabezado del archivo "onu_ingles.csv" para la creación de la tabla de datos.

CREATE TABLE csv_onu_ingles(
   global_code                char(3),
   global_name                varchar(250),
   codigo_region              char(3),
   nombre_region_ingles       varchar(30),
   codigo_sub_region          char(3),
   nombre_subregion_ingles    varchar(40),
   intermediate_region_code   varchar(250),
   intermediate_region_name   varchar(250),
   nombre_ingles              varchar(60),
   m49                        char(3),
   iso3166_3                  char(3),
   ldc                        char(1),
   lldc                       char(1),
   sids                       char(1),
   estatus_desarrollo         varchar(15)
); 

Importamos el contenido del archivo onu_ingles.csv

# \copy csv_onu_ingles FROM '/ruta_al_archivo/.../onu_ingles.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 
Tabla ONU Chino

Obtenemos el encabezado del archivo "onu_chino.csv" para la creación de la tabla de datos.

CREATE TABLE csv_onu_chino(
   global_code                char(3),
   global_name                varchar(250),
   codigo_region              char(3),
   nombre_region_chino        varchar(30),
   codigo_sub_region          char(3),
   nombre_subregion_chino     varchar(40),
   intermediate_region_code   varchar(250),
   intermediate_region_name   varchar(250),
   nombre_chino               varchar(60),
   m49                        char(3),
   iso3166_3                  char(3),
   ldc                        char(1),
   lldc                       char(1),
   sids                       char(1),
   estatus_desarrollo         varchar(15)
); 

Importamos el contenido del archivo onu_chino.csv

# \copy csv_onu_chino FROM '/ruta_al_archivo/.../onu_chino.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 
Tabla ONU Ruso

Obtenemos el encabezado del archivo "onu_ruso.csv" para la creación de la tabla de datos.

CREATE TABLE csv_onu_ruso(
   global_code                char(3),
   global_name                varchar(250),
   codigo_region              char(3),
   nombre_region_ruso         varchar(30),
   codigo_sub_region          char(3),
   nombre_subregion_ruso      varchar(40),
   intermediate_region_code   varchar(250),
   intermediate_region_name   varchar(250),
   nombre_ruso                varchar(60),
   m49                        char(3),
   iso3166_3                  char(3),
   ldc                        char(1),
   lldc                       char(1),
   sids                       char(1),
   estatus_desarrollo         varchar(15)
); 

Importamos el contenido del archivo onu_ruso.csv

# \copy csv_onu_ruso FROM '/ruta_al_archivo/.../onu_ruso.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 
Tabla ONU Francés

Obtenemos el encabezado del archivo "onu_frances.csv" para la creación de la tabla de datos.

CREATE TABLE csv_onu_frances(
   global_code                char(3),
   global_name                varchar(250),
   codigo_region              char(3),
   nombre_region_frances      varchar(30),
   codigo_sub_region          char(3),
   nombre_subregion_frances   varchar(40),
   intermediate_region_code   varchar(250),
   intermediate_region_name   varchar(250),
   nombre_frances             varchar(60),
   m49                        char(3),
   iso3166_3                  char(3),
   ldc                        char(1),
   lldc                       char(1),
   sids                       char(1),
   estatus_desarrollo         varchar(15)
); 

Importamos el contenido del archivo onu_frances.csv

# \copy csv_onu_frances FROM '/ruta_al_archivo/.../onu_frances.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 
Tabla ONU Árabe

Obtenemos el encabezado del archivo "onu_arabe.csv" para la creación de la tabla de datos.

CREATE TABLE csv_onu_arabe(
   global_code                char(3),
   global_name                varchar(250),
   codigo_region              char(3),
   nombre_region_arabe        varchar(30),
   codigo_sub_region          char(3),
   nombre_subregion_arabe     varchar(40),
   intermediate_region_code   varchar(250),
   intermediate_region_name   varchar(250),
   nombre_arabe               varchar(60),
   m49                        char(3),
   iso3166_3                  char(3),
   ldc                        char(1),
   lldc                       char(1),
   sids                       char(1),
   estatus_desarrollo         varchar(15)
); 

Importamos el contenido del archivo onu_arabe.csv

# \copy csv_onu_arabe FROM '/ruta_al_archivo/.../onu_arabe.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 
Tabla Información Adicional

En el archivo "country_codes.csv"  se encuentra mucha de la información adicional para nuestra tabla de paises, por lo que obtenemos su encabezado y creamos la tabla.

CREATE TABLE csv_informacion_paises(
   name                             varchar(256),
   official_name_en                 varchar(256),
   official_name_fr                 varchar(256),
   iso3166_2                        char(2),
   iso3166_3                        char(3),
   M49                              smallint,
   itu                              char(3),
   MARC                             varchar(256),
   wmo                              char(2),
   DS                               varchar(256),
   dial                             varchar(20),
   fifa                             varchar(20),
   FIPS                             varchar(256),
   GAUL                             varchar(256),
   ioc                              char(3),
   codigo_alfabetico_moneda         char(3),
   ISO4217_currency_country_name    varchar(256),
   ISO4217_currency_minor_unit      smallint,
   nombre_moneda_ingles             varchar(50),
   codigo_numerico_moneda           smallint,
   is_independent                   varchar(100),
   Capital                          varchar(100),
   Continent                        varchar(100),
   tld                              char(3),
   lenguajes                        varchar(100),
   Geoname_ID                       integer,
   edgar                            char(2)
);

Importamos el contenido del archivo country_codes.csv

# \copy csv_informacion_paises FROM '/ruta_al_archivo/.../country_codes.csv' DELIMITER ',' QUOTE '"' CSV HEADER;

 

 

 

 
Entregable 1

\copy (SELECT o.iso3166_1, o.iso3166_2, o.iso3166_3, o.nombre_espanol, o.nombre_ingles, o.nombre_chino, o.nombre_ruso, o.nombre_frances, o.nombre_arabe, o.latitud, o.longitud, o.m49, o.estatus_desarrollo, o.codigo_region, o.codigo_sub_region, o.nombre_region_espanol, o.nombre_region_ingles, o.nombre_region_chino, o.nombre_region_ruso, o.nombre_region_frances, o.nombre_region_arabe, o.nombre_subregion_espanol, o.nombre_subregion_ingles, o.nombre_subregion_chino, o.nombre_subregion_ruso, o.nombre_subregion_frances, o.nombre_subregion_arabe, p.itu, p.wmo, p.dial, p.fifa, p.ioc, p.tld, p.lenguajes, p.codigo_alfabetico_moneda, p.nombre_moneda_ingles, p.codigo_numerico_moneda,p.edgar FROM (SELECT m.*, n.nombre_arabe, n.nombre_region_arabe, n.nombre_subregion_arabe FROM (SELECT k.*, l.nombre_frances, l.nombre_region_frances, l.nombre_subregion_frances FROM (SELECT i.*, j.nombre_ruso, j.nombre_region_ruso, j.nombre_subregion_ruso FROM (SELECT g.*, h.nombre_chino, h.nombre_region_chino, h.nombre_subregion_chino FROM (SELECT e.*, f.nombre_ingles, f.nombre_region_ingles, f.nombre_subregion_ingles FROM (SELECT c.*, d.nombre_espanol, d.codigo_region, d.codigo_sub_region, d.nombre_region_espanol, d.nombre_subregion_espanol, d.m49, d.estatus_desarrollo FROM (SELECT a.iso3166_1, a.iso3166_2, a.iso3166_3, b.latitud, b.longitud FROM csv_iso3166 a LEFT JOIN csv_latitud_longitud b ON a.iso3166_2 = b.iso3166_2) as c LEFT JOIN csv_onu_espanol d ON c.iso3166_3 = d.iso3166_3) as e LEFT JOIN csv_onu_ingles f ON e.iso3166_3 = f.iso3166_3) as g LEFT JOIN csv_onu_chino h ON g.iso3166_3 = h.iso3166_3) as i LEFT JOIN csv_onu_ruso j ON i.iso3166_3 = j.iso3166_3) as k LEFT JOIN csv_onu_frances l ON k.iso3166_3 = l.iso3166_3) as m LEFT JOIN csv_onu_arabe n ON m.iso3166_3 = n.iso3166_3) as o LEFT JOIN csv_informacion_paises p ON o.iso3166_3 = p.iso3166_3 ORDER BY o.iso3166_1) TO '/ruta_en_el_equipo_local/.../base_paises_final.csv' WITH(DELIMITER '|', HEADER, FORMAT csv, QUOTE '"', FORCE_QUOTE *);

 
La consulta

Para desplegar los datos de los paises conforme se solicita en el diccionario de datos utilizando tablas creadas con los archivos CSV

SELECT o.iso3166_1, o.iso3166_2, o.iso3166_3, o.nombre_espanol, o.nombre_ingles, o.nombre_chino, o.nombre_ruso, o.nombre_frances, o.nombre_arabe, o.latitud, o.longitud, o.m49, o.estatus_desarrollo, o.codigo_region, o.codigo_sub_region, o.nombre_region_espanol, o.nombre_region_ingles, o.nombre_region_chino, o.nombre_region_ruso, o.nombre_region_frances, o.nombre_region_arabe, o.nombre_subregion_espanol, o.nombre_subregion_ingles, o.nombre_subregion_chino, o.nombre_subregion_ruso, o.nombre_subregion_frances, o.nombre_subregion_arabe, p.itu, p.wmo, p.dial, p.fifa, p.ioc, p.tld, p.lenguajes, p.codigo_alfabetico_moneda, p.nombre_moneda_ingles, p.codigo_numerico_moneda,p.edgar FROM 
(SELECT m.*, n.nombre_arabe, n.nombre_region_arabe, n.nombre_subregion_arabe FROM 
(SELECT k.*, l.nombre_frances, l.nombre_region_frances, l.nombre_subregion_frances FROM 
(SELECT i.*, j.nombre_ruso, j.nombre_region_ruso, j.nombre_subregion_ruso FROM 
(SELECT g.*, h.nombre_chino, h.nombre_region_chino, h.nombre_subregion_chino FROM 
(SELECT e.*, f.nombre_ingles, f.nombre_region_ingles, f.nombre_subregion_ingles FROM 
(SELECT c.*, d.nombre_espanol, d.codigo_region, d.codigo_sub_region, d.nombre_region_espanol, d.nombre_subregion_espanol, d.m49, d.estatus_desarrollo FROM 
(SELECT a.iso3166_1, a.iso3166_2, a.iso3166_3, b.latitud, b.longitud FROM 
csv_iso3166 a LEFT JOIN csv_latitud_longitud b ON a.iso3166_2 = b.iso3166_2) 
as c LEFT JOIN csv_onu_espanol d ON c.iso3166_3 = d.iso3166_3) 
as e LEFT JOIN csv_onu_ingles f ON e.iso3166_3 = f.iso3166_3) 
as g LEFT JOIN csv_onu_chino h ON g.iso3166_3 = h.iso3166_3) 
as i LEFT JOIN csv_onu_ruso j ON i.iso3166_3 = j.iso3166_3) 
as k LEFT JOIN csv_onu_frances l ON k.iso3166_3 = l.iso3166_3)
as m LEFT JOIN csv_onu_arabe n ON m.iso3166_3 = n.iso3166_3) 
as o LEFT JOIN csv_informacion_paises p ON o.iso3166_3 = p.iso3166_3;

 
Entregable 2

CREATE TABLE cat_paises(
   iso3166_1                  char(3),
   iso3166_2                  char(2),
   iso3166_3                  char(3),
   nombre_espanol             varchar(60),
   nombre_ingles              varchar(60),
   nombre_chino               varchar(60),
   nombre_ruso                varchar(60),
   nombre_frances             varchar(60),
   nombre_arabe               varchar(60),
   latitud                    float8,
   longitud                   float8,
   m49                        char(3),
   estatus_desarrollo         varchar(15),
   codigo_region              char(3),
   codigo_sub_region          char(3),
   nombre_region_espanol      varchar(30),
   nombre_region_ingles       varchar(30),
   nombre_region_chino        varchar(30),
   nombre_region_ruso         varchar(30),
   nombre_region_frances      varchar(30),
   nombre_region_arabe        varchar(30),
   nombre_subregion_espanol   varchar(40),
   nombre_subregion_ingles    varchar(40),
   nombre_subregion_chino     varchar(40),
   nombre_subregion_ruso      varchar(40),
   nombre_subregion_frances   varchar(40),
   nombre_subregion_arabe     varchar(40),
   itu 	                      char(3),
   wmo                        char(2),
   dial                       varchar(20),
   fifa                       varchar(20),
   ioc                        char(3),
   tld                        char(3),
   lenguajes                  varchar(100),
   codigo_alfabetico_moneda   char(3),
   nombre_moneda_ingles       varchar(50),
   codigo_numerico_moneda     smallint,
   edgar                      char(2)
);

 
Entregable 3

La carga de la información a la tabla puede ser de tres formas, pasando la información tabla por tabla, del archivo generado anteriormente, o directamente desde la consulta.

INSERT INTO cat_paises SELECT o.iso3166_1, o.iso3166_2, o.iso3166_3, o.nombre_espanol, o.nombre_ingles, o.nombre_chino, o.nombre_ruso, o.nombre_frances, o.nombre_arabe, o.latitud, o.longitud, o.m49, o.estatus_desarrollo, o.codigo_region, o.codigo_sub_region, o.nombre_region_espanol, o.nombre_region_ingles, o.nombre_region_chino, o.nombre_region_ruso, o.nombre_region_frances, o.nombre_region_arabe, o.nombre_subregion_espanol, o.nombre_subregion_ingles, o.nombre_subregion_chino, o.nombre_subregion_ruso, o.nombre_subregion_frances, o.nombre_subregion_arabe, p.itu, p.wmo, p.dial, p.fifa, p.ioc, p.tld, p.lenguajes, p.codigo_alfabetico_moneda, p.nombre_moneda_ingles, p.codigo_numerico_moneda,p.edgar FROM (SELECT m.*, n.nombre_arabe, n.nombre_region_arabe, n.nombre_subregion_arabe FROM (SELECT k.*, l.nombre_frances, l.nombre_region_frances, l.nombre_subregion_frances FROM (SELECT i.*, j.nombre_ruso, j.nombre_region_ruso, j.nombre_subregion_ruso FROM (SELECT g.*, h.nombre_chino, h.nombre_region_chino, h.nombre_subregion_chino FROM (SELECT e.*, f.nombre_ingles, f.nombre_region_ingles, f.nombre_subregion_ingles FROM (SELECT c.*, d.nombre_espanol, d.codigo_region, d.codigo_sub_region, d.nombre_region_espanol, d.nombre_subregion_espanol, d.m49, d.estatus_desarrollo FROM (SELECT a.iso3166_1, a.iso3166_2, a.iso3166_3, b.latitud, b.longitud FROM csv_iso3166 a LEFT JOIN csv_latitud_longitud b ON a.iso3166_2 = b.iso3166_2) as c LEFT JOIN csv_onu_espanol d ON c.iso3166_3 = d.iso3166_3) as e LEFT JOIN csv_onu_ingles f ON e.iso3166_3 = f.iso3166_3) as g LEFT JOIN csv_onu_chino h ON g.iso3166_3 = h.iso3166_3) as i LEFT JOIN csv_onu_ruso j ON i.iso3166_3 = j.iso3166_3) as k LEFT JOIN csv_onu_frances l ON k.iso3166_3 = l.iso3166_3) as m LEFT JOIN csv_onu_arabe n ON m.iso3166_3 = n.iso3166_3) as o LEFT JOIN csv_informacion_paises p ON o.iso3166_3 = p.iso3166_3 ORDER BY o.iso3166_1;
