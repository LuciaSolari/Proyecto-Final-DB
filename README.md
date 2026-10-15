# TRABAJO FINAL DE BASE DE DATOS Y DE GESTIÓN DE PROYECTOS DE SOFTWARE #

# ESCUELA SUPERIOR DE COMERCIO Nº 43 #

# CARRERA: TÉCNICO SUPERIOR EN DESARROLLO DE SOFTWARE #

# PROFESORES: ARIEL NARDELLI, MATIAS NARDELLI, ELIANA SULIGOY #

# ALUMNOS: OMAR OJEDA, LUCÍA SOLARI #

## EL CVS QUE ELEGIMOS FUE UN DATASET DE PELÍCULAS Y SERIES DE NETFLIX ##
## El dataset que elegimos tiene más de 8.000 registros de películas y series, las tablas que el dataset original tenía eran:
## show_id, type, title, director, cast, country, date_added, release_year, rating, duration, listen_in y description. 
## Íbamos viendo cuáles podíamos agrupar y normalizar, también en vez de dejarlas en inglés, las cambiamos a español.

#** DIAGRAMA ER **#


** CÓDIGO UTILIZADO PARA LA NORMALIZACIÓN **

### Reemplazar en un documento de texto el archivo exportado para que a la hora de importar no haya muchos problemas.
### Reemplazar todos: utf8mb4_0900_ai_ci
### Por: utf8mb4_unicode_ci

** CONSULTAS REALIZADAS **

####-- --------------------------------------------------
####-- Nueva base de datos limpia | proyectoFinalDB_v2
####-- --------------------------------------------------
#### CREATE DATABASE proyectoFinalDB_v2;
#### USE proyectoFinalDB_v2;
####
####-- ------------------------------------ --
####		-- CREAR LA TABLA DATASET --
####-- ------------------------------------ --
#### CREATE TABLE dataset(
####  id_show INT AUTO_INCREMENT NOT NULL,
####  TYPE TEXT,
####	title TEXT NOT NULL,
####	director TEXT,
####	CAST TEXT,
####	country TEXT,
####	date_added TEXT,
####	release_year INT,
####	rating TEXT,
####	duration TEXT,
####	listed_in TEXT,
####	DESCRIPTION TEXT,
####   PRIMARY KEY (id_show)
#### ) ENGINE=InnoDB
#### DEFAULT CHARSET=UTF8MB4;

####-- ------------------------------------ --
####-- CARGAR LOS DATOS DEL CSV EN LA TABLA --
####-- ------------------------------------ --
#### LOAD DATA INFILE
#### '/var/lib/mysql-files/netflix_titles.csv'
#### INTO TABLE dataset
#### FIELDS TERMINATED BY ','
#### OPTIONALLY ENCLOSED BY '"'
#### LINES TERMINATED BY '\n'
#### IGNORE 1 ROWS
#### (
####   @id_show,
####   TYPE,
####   title,
####   director,
####   CAST,
####   country,
####   date_added,
####   release_year,
####   rating,
####   duration,
####   listed_in,
####   DESCRIPTION
#### );

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO --
####-- ------------------------------------ --
#### SELECT * FROM dataset;

####-- ------------------------------------ --
####		-- ANOTACIONES --
####-- ------------------------------------ --
####-- HAY 8807 REGISTROS DE PELICULAS EN EL DATASET

####-- ------------------------------------ --
####		-- CREAR TABLA TIPOS --
####-- ------------------------------------ --
#### CREATE TABLE tipos(
####	id_tipo INT NOT NULL AUTO_INCREMENT,
####	tipo_de_show VARCHAR(20) NOT NULL,	
####	  PRIMARY KEY (id_tipo)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;

####-- ------------------------------------ --
####		-- INSERTS EN LA TABLA TIPOS --
####-- ------------------------------------ --
#### INSERT INTO tipos(tipo_de_show)
#### SELECT DISTINCT
####	type
#### FROM dataset;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: CANTIDADES POR TIPO --
####-- ------------------------------------ --
#### SELECT COUNT(*),type FROM dataset
#### GROUP BY TYPE;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE TIPOS --
####-- ------------------------------------ --
#### SELECT * FROM tipos;

####-- ------------------------------------ --
####-- ANOTACIONES VARIAS: --
####-- ------------------------------------ --
#### HAY 6131 PELICULAS Y 2676 TV SHOWS

####-- ------------------------------------ --
####-- CREAR TABLA: CLASIFICACIONES --
####-- ------------------------------------ --
#### CREATE TABLE clasificaciones(
####	id_clasificacion INT NOT NULL AUTO_INCREMENT,
####	clasificacion VARCHAR(20) NOT NULL,
####		PRIMARY KEY(id_clasificacion)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;

####-- ------------------------------------ --
####-- INSERTS EN LA TABLA: CLASIFICACIONES --
####-- ------------------------------------ --
#### INSERT INTO clasificaciones(clasificacion)
#### SELECT DISTINCT rating FROM dataset
#### WHERE rating <> '' AND rating NOT LIKE '%min%';

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODOS LOS RATINGS DEL DATASET CON DISTINCT--
####-- ------------------------------------ --
#### SELECT DISTINCT rating FROM dataset;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODAS LAS CLASIFICACIONES DE DATASET, DESCARTANDO LAS VACIAS Y CON MINS--
####-- ------------------------------------ --
#### SELECT DISTINCT rating FROM dataset
#### WHERE rating <> '' AND rating NOT LIKE '%min%';

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: VECES QUE RATING APARECE VACIO EN EL DATASET (4)--
####-- ------------------------------------ --
#### SELECT DISTINCT COUNT(*), rating FROM dataset
#### WHERE rating = '';

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: CANTIDAD DE VECES QUE SE REPITEN LAS CATEGORIAS EN EL DATASET--
####-- ------------------------------------ --
#### SELECT COUNT(*) AS apariciones, rating FROM dataset
#### WHERE rating <> '' AND rating NOT LIKE '%min%'
#### GROUP BY rating
#### ORDER BY apariciones DESC;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: CANTIDAD DE APARICIONES REPETIDAS --
####-- ------------------------------------ --
#### SELECT COUNT(*) AS apariciones FROM(
#### SELECT rating FROM dataset
#### WHERE rating <> '' AND rating NOT LIKE '%min%'
#### ) AS apariciones;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE CLASIFICACIONES --
####-- ------------------------------------ --
#### SELECT * FROM clasificaciones;

####-- ------------------------------------ --
####-- ANOTACIONES VARIAS: --
####-- ------------------------------------ --
#### EN EL DATASET RATING TIENE 4 VECES VACIO Y 3 DATOS CON MINUTOS
#### HAY 14 CLASIFICACIONES DISTINTAS EN TOTAL, 8800 EN EL DATASET (QUITANDO LAS VACIAS Y CORRUPTAS)

####-- ------------------------------------ --
####		-- CREAR TABLA: SHOWS --
####-- ------------------------------------ --
#### CREATE TABLE shows(
####	id_show INT NOT NULL AUTO_INCREMENT,
####	id_tipo INT NOT NULL,
####	titulo VARCHAR(150) NOT NULL,
####	fecha_aniadido TEXT NULL,
####	anio_salida INT NOT NULL,
####	id_clasificacion INT NULL,
####	duracion VARCHAR(20) NOT NULL,
####	descripcion VARCHAR(300) NOT NULL,
####		PRIMARY KEY(id_show),
####		FOREIGN KEY(id_tipo)
####	REFERENCES tipos (id_tipo),
####  FOREIGN KEY(id_clasificacion)
####  REFERENCES clasificaciones (id_clasificacion)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;

####-- ------------------------------------ --
####-- INSERTS EN LA TABLA:  --
####-- ------------------------------------ --
#### INSERT INTO shows(id_tipo,titulo,fecha_aniadido,anio_salida,id_clasificacion,duracion,descripcion)
#### SELECT
####	t.id_tipo,
####	d.title,
####	d.date_added,
####	d.release_year,
####	c.id_clasificacion,
####	d.duration,
####	d.DESCRIPTION
#### FROM dataset AS d
#### JOIN tipos AS t ON t.tipo_de_show = d.type
#### LEFT JOIN clasificaciones AS c ON c.clasificacion = d.rating;

####-- ------------------------------------ --
####-- CAMBIANDO EL FORMATO DE LA COLUMNA FECHA_ANIADIDO A DATE:  --
####-- ------------------------------------ --
#### UPDATE shows
#### SET fecha_aniadido = STR_TO_DATE(fecha_aniadido, '%M %d, %Y')
#### WHERE fecha_aniadido IS NOT NULL AND fecha_aniadido <> '';

#### ALTER TABLE shows MODIFY COLUMN fecha_aniadido TEXT NULL;

#### UPDATE shows
#### SET fecha_aniadido = NULL
#### WHERE fecha_aniadido = '';

#### ALTER TABLE shows
#### MODIFY fecha_aniadido DATE;

#### -- ------------------------------------ --
####-- CONSULTAS VARIAS: VER EL VALOR CON MAS CARACTERES EN LA TABLA DEL DATASET--
####-- ------------------------------------ --
#### SELECT title FROM dataset
#### WHERE CHAR_LENGTH(title) > 100;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: ID_SHOW + ID_TIPO + NOMBRE DEL TIPO SHOW--
####-- ------------------------------------ --
#### SELECT
####	d.id_show,
####	t.id_tipo,
####	t.tipo_de_show
#### FROM dataset AS d
#### JOIN tipos AS t ON t.tipo_de_show = d.type;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: MUY PARECIDA A LA ORIGINAL--
####-- ------------------------------------ --
#### SELECT
####	d.id_show,
####	t.tipo_de_show,
####	d.title,
####	d.date_added,
####	d.release_year,
####	c.clasificacion,
####	d.duration,
####	d.DESCRIPTION
#### FROM dataset AS d
#### JOIN tipos AS t ON t.tipo_de_show = d.type
#### LEFT JOIN clasificaciones AS c ON c.clasificacion = d.rating;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: ver cantidad de veces que aparece anio_salidaL--
####-- ------------------------------------ --
#### SELECT * FROM shows
#### WHERE fecha_aniadido = '';

#### SELECT COUNT(anio_salida) FROM shows;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: CHEQUEAR LAS CLASIFICACIONES EN DATASET QUE SEAN NULL--
####-- ------------------------------------ --
#### SELECT d.id_show, d.rating
#### FROM dataset AS d
#### LEFT JOIN clasificaciones AS c ON c.clasificacion = d.rating
#### WHERE c.id_clasificacion IS NULL;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE SHOWS--
####-- ------------------------------------ --
#### SELECT * FROM shows

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: VER LOS SHOWS CUYA FECHA_ANIADIDO SEA LA MAS VIEJA--
####-- ------------------------------------ --
#### SELECT MIN(YEAR(fecha_aniadido)) FROM shows;

#### SELECT * FROM shows
#### WHERE YEAR(fecha_aniadido) = (
####	SELECT MIN(YEAR(fecha_aniadido))FROM shows
#### );

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: PRUEBA DE SIMULACION AL DATASET ORIGINAL --
####-- ------------------------------------ --
#### SELECT 
####	s.id_show,
####	t.tipo_de_show,
####	s.titulo,
####	d.id_director,
####	d.nombre_director,
####	s.fecha_aniadido, 
####	s.anio_salida,
####	c.clasificacion,
####	s.duracion,
####	s.descripcion
#### FROM shows AS s
#### LEFT JOIN shows_directores AS sd ON sd.id_show = s.id_show
#### LEFT JOIN directores AS d ON d.id_director = sd.id_director
#### LEFT JOIN tipos AS t ON t.id_tipo = s.id_tipo
#### LEFT JOIN clasificaciones AS c ON c.id_clasificacion = s.id_clasificacion
#### ORDER BY s.id_show;

####-- ------------------------------------ --
####-- CREAR TABLA DIRECTORES --
####-- ------------------------------------ --
#### CREATE TABLE directores(
#### 	id_director INT NOT NULL AUTO_INCREMENT,
#### 	nombre_director VARCHAR(100) NOT NULL,
####	PRIMARY KEY(id_director)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;

####-- ------------------------------------ --
####-- INSERTS EN LA TABLA DIRECTORES --
####-- ------------------------------------ --
#### INSERT INTO directores(nombre_director)
#### SELECT DISTINCT 
####	TRIM(j.nombre_director) AS director
#### FROM dataset AS d
#### JOIN JSON_TABLE(
####	CONCAT(
####		'["',REPLACE(REPLACE(d.director,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_director VARCHAR(100) PATH '$'
####	)
#### )AS j
#### WHERE TRIM(d.director) <> '' AND TRIM(j.nombre_director) <> '';
####-- AND TRIM(j.nombre_director) LIKE '%%';
####-- CONSULTAS VARIAS: CANTIDAD DE DIRECTORES UNICOS --
####-- PARA DIRECTORES REPETIDOS QUITAR EL DISTINCT

#### SELECT COUNT(*) AS cantidad_directores FROM(
#### SELECT DISTINCT 
####	TRIM(j.nombre_director) AS director
####-- ----------------------------------------------- --
####-- CONSULTAS VARIAS: CANTIDAD DE DIRECTORES VACIOS --
####-- ----------------------------------------------- --
#### SELECT DISTINCT COUNT(*), director FROM dataset
#### WHERE director = ''
#### GROUP BY todo;

#### FROM dataset AS d
#### JOIN JSON_TABLE(
####	CONCAT(
####		'["',REPLACE(REPLACE(d.director,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_director VARCHAR(100) PATH '$'
####	)
#### )AS j
#### WHERE TRIM(d.director) <> '' AND TRIM(j.nombre_director) <> ''
#### ) AS directore;

####-- ----------------------------------------------------- --
####-- CONSULTAS VARIAS: CANTIDAD DE APARICION DE DIRECTORES --
####-- ----------------------------------------------------- --
#### SELECT 
####	TRIM(j.nombre_director) AS director,
####	COUNT(*) AS apariciones
#### FROM dataset AS d
#### JOIN JSON_TABLE(
####	CONCAT(
####		'["',REPLACE(REPLACE(d.director,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_director VARCHAR(100) PATH '$'
####	)
#### )AS j
#### WHERE TRIM(d.director) <> '' AND TRIM(j.nombre_director) <> ''
#### GROUP BY TRIM(j.nombre_director)
#### ORDER BY apariciones DESC;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE DIRECTORES --
####-- ------------------------------------ --
#### SELECT COUNT(*) FROM directores;

####-- ----------------------------------------------------------------------- --
####-- CONSULTAS VARIAS: directores cuyos nombres sean mayores a 20 caracteres --
####-- ----------------------------------------------------------------------- --
#### SELECT nombre_director FROM directores
#### WHERE CHAR_LENGTH(nombre_director) > 20;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE DIRECTORES --
####-- ------------------------------------ --
#### SELECT * FROM directores;

####-- ------------------------------------ --
####-- ANOTACIONES VARIAS: --
####-- ------------------------------------ --
####-- HAY 2634 DIRECTORES VACIOS EN DATASET
####-- NO HAY NULOS
####-- HAY 6762 DIRECTORES (ALGUNOS REPETIDOS)
####-- HAY 4814 DIRECTORES (INDIVIDUALES)
####-- EL DIRECTOR QUE MAS APARECIO FUE RAJIV CHILAKA (22)

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE DIRECTORES --
####-- ------------------------------------ --
#### SELECT COUNT(*) FROM directores;

####-- ----------------------------------------------------------------------- --
####-- CONSULTAS VARIAS: directores cuyos nombres sean mayores a 20 caracteres --
####-- ----------------------------------------------------------------------- --
#### SELECT nombre_director FROM directores
#### WHERE CHAR_LENGTH(nombre_director) > 20;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE DIRECTORES --
####-- ------------------------------------ --
#### SELECT * FROM directores;

####-- ------------------------------------ --
####-- ANOTACIONES VARIAS: --
####-- ------------------------------------ --
####-- HAY 2634 DIRECTORES VACIOS EN DATASET
####-- NO HAY NULOS
####-- HAY 6762 DIRECTORES (ALGUNOS REPETIDOS)
####-- HAY 4814 DIRECTORES (INDIVIDUALES)
####-- EL DIRECTOR QUE MAS APARECIO FUE RAJIV CHILAKA (22)

####-- ----------------------------- --
####-- CREAR TABLA: SHOWS_DIRECTORES --
####-- ----------------------------- --
#### CREATE TABLE shows_directores(
####	id_show INT NOT NULL,
####	id_director INT NOT NULL,
####	PRIMARY KEY(id_show,id_director),
#### FOREIGN KEY(id_show)
#### REFERENCES shows(id_show),
#### FOREIGN KEY(id_director)
####	REFERENCES directores(id_director)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;
#### SHOW TABLES;

####-- ------------------------------------- --
####-- INSERTS EN LA TABLA: SHOWS_DIRECTORES --
####-- ------------------------------------- --
#### INSERT INTO shows_directores(id_show, id_director)
#### SELECT DISTINCT
####	d.id_show,
####	di.id_director
#### FROM dataset AS d 
#### JOIN JSON_TABLE(
#### 	CONCAT(
####		'["',REPLACE(REPLACE(d.director,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_director VARCHAR(100) PATH '$'
####	)
#### ) AS j
#### JOIN directores AS di ON di.nombre_director = TRIM(j.nombre_director)
#### WHERE TRIM(d.director) <> '' AND TRIM(j.nombre_director) <> '';

####-- -------------------- --
####-- CREAR TABLA: ACTORES --
####-- -------------------- --
#### CREATE TABLE actores(
####	id_actor INT NOT NULL AUTO_INCREMENT,
####	nombre_actor VARCHAR(100) NOT NULL,
	
####	PRIMARY KEY(id_actor)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;

####-- ---------------------------- --
####-- INSERTS EN LA TABLA: ACTORES --
####-- ---------------------------- --
#### INSERT INTO actores(nombre_actor)
#### SELECT DISTINCT
#### 	TRIM(j.nombre_actor)
#### FROM dataset AS d
#### JOIN JSON_TABLE(
####	CONCAT(
####		'["',REPLACE(REPLACE(d.`CAST`,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_actor VARCHAR(100) PATH '$'
####	)
#### ) AS j
#### WHERE TRIM(d.`CAST`) <> '' AND TRIM(j.nombre_actor) <> ''
#### AND TRIM(j.nombre_actor) NOT IN(
####	'Cast members of the "To All the Boys" films dish about the third movie and answer some fan questions',
####	'plus Whitney Cummings gives suspect dating advice.',
####	'Craig Robinson & The Nasty Delicious',
####	'Casting a light on the mistreatment of sea mammals',
####	'this documentary follows the plight of 18 Beluga whales kept in captivity for years.'
#### );

####-- --------------------------- --
####-- CREAR TABLA SHOWS_ACTORES:  --
####-- ------------- ------------- --
#### CREATE TABLE shows_actores(
####	id_show INT NOT NULL,
####	id_actor INT NOT NULL,
####		PRIMARY KEY(id_show,id_actor),
####	FOREIGN KEY(id_show)
####	REFERENCES shows(id_show),
#### FOREIGN KEY(id_actor)
####	REFERENCES actores(id_actor)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;
#### SHOW TABLES;

####-- ------------------------------------ --
####-- INSERTS EN LA TABLA SHOWS_ACTORES:  --
####-- ------------------------------------ --
#### INSERT INTO shows_actores(id_show, id_actor)
#### SELECT DISTINCT
####	d.id_show,
####	a.id_actor
#### FROM dataset AS d 
#### JOIN JSON_TABLE(
####	CONCAT(
####		'["',REPLACE(REPLACE(d.`CAST`,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_actor VARCHAR(100) PATH '$'
####	)
#### ) AS j
#### JOIN actores AS a ON a.nombre_actor = TRIM(j.nombre_actor)
#### WHERE TRIM(d.`CAST`) <> '' AND TRIM(j.nombre_actor) <> '';

####-- ----------------- --
####-- CONSULTAS VARIAS: --
####-- ----------------- --
#### SELECT * FROM shows_actores
#### ORDER BY id_show;

####-- ------------------------------------ --
####-- CONSULTAS VARIAS: TODO DE ACTORES --
####-- ------------------------------------ --
#### SELECT * FROM actores
#### -- WHERE id_actor IN (5110, 28776, 28775, 5111, 24182);

####-- ------------------------------------------------------ --
####-- CONSULTAS VARIAS: ACTORES CON TOTAL DE CARACTERES DESC --
####-- ------------------------------------------------------ --
#### SELECT id_actor, CHAR_LENGTH(nombre_actor) AS p FROM actores
#### ORDER BY p DESC;

####-- ----------------- --
####-- CONSULTAS VARIAS: --
####-- ------------------ --
#### SELECT * FROM shows_directores;

####-- ------------------------------------ --
####-- CREAR TABLA SHOWS_PAISES:  --
####-- ------------------------------------ --
#### CREATE TABLE shows_paises(
####	id_show INT NOT NULL,
####	id_pais INT NOT NULL,
####	PRIMARY KEY(id_show,id_pais),	
####	FOREIGN KEY(id_show)
####	REFERENCES shows(id_show),
#### FOREIGN KEY(id_pais)
####	REFERENCES paises(id_pais)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;
#### SHOW TABLES;

####-- ---------------------------------- --
####-- INSERTS EN LA TABLA SHOWS_PAISES:  --
####-- ---------------------------------- --
#### INSERT INTO shows_paises(id_show, id_pais)
#### SELECT DISTINCT
####	d.id_show,
####	p.id_pais
#### FROM dataset AS d 
#### JOIN JSON_TABLE(
####	CONCAT(
####		'["',REPLACE(REPLACE(d.country,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_pais VARCHAR(100) PATH '$'
####	)
#### ) AS j
#### JOIN paises AS p ON p.nombre_pais = TRIM(j.nombre_pais)
#### WHERE TRIM(d.country) <> '' AND TRIM(j.nombre_pais) <> '';

####-- ----------------------------- --
####-- CREAR TABLA: SHOWS_CATEGORIAS --
####-- ----------------------------- --
#### CREATE TABLE shows_categorias(
####	id_show INT NOT NULL,
####	id_categoria INT NOT NULL,
#### PRIMARY KEY(id_show, id_categoria),
####	FOREIGN KEY (id_show)
####	REFERENCES shows (id_show),	
####	FOREIGN KEY (id_categoria)
####	REFERENCES categorias (id_categoria)
#### )ENGINE=INNODB
#### DEFAULT CHARSET=UTF8MB4;
#### SHOW TABLES;

####-- ------------------------------------ --
####-- INSERTS EN LA TABLA: SHOWS CATEGORIAS --
####-- ------------------------------------ --
#### INSERT INTO shows_categorias(id_show,id_categoria)
#### SELECT DISTINCT
####	d.id_show,
####	c.id_categoria
#### FROM dataset AS d
#### JOIN JSON_TABLE(
####	CONCAT(
####		'["',REPLACE(REPLACE(d.listed_in,'"','\\"'),', ','","'),'"]'
####	),
####	'$[*]' COLUMNS(
####		nombre_categoria VARCHAR(50) PATH '$'
####	)
#### ) AS j
#### JOIN categorias AS c ON c.categoria = TRIM(j.nombre_categoria)
#### WHERE TRIM(d.listed_in) <> '' AND TRIM(j.nombre_categoria) <> ''
#### ORDER BY id_show;

-- ------------------------------------ --
-- CONSULTAS VARIAS: Todo Shows categorias --
-- ------------------------------------ --
SELECT * FROM shows_categorias
ORDER BY id_show;












