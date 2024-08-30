CREATE SCHEMA `prueba_universidad` ;

--Creando las Tablas

CREATE TABLE `prueba_universidad`.`estudiante` (
  `estudiante_id` INT NOT NULL AUTO_INCREMENT,
  `estudiante_nombre` VARCHAR(25) NOT NULL,
  `estudiante_apellido` VARCHAR(25) NOT NULL,
  `estudiante_email` VARCHAR(80) NULL,
  UNIQUE INDEX `estudiante_id_UNIQUE` (`estudiante_id` ASC) VISIBLE,
  UNIQUE INDEX `estudiante_email_UNIQUE` (`estudiante_email` ASC) VISIBLE,
  PRIMARY KEY (`estudiante_id`));

CREATE TABLE `prueba_universidad`.`profesor` (
  `profesor_id` INT NOT NULL AUTO_INCREMENT,
  `profesor_nombre` VARCHAR(20) NOT NULL,
  `profesor_apellido` VARCHAR(20) NOT NULL,
  `departamento` VARCHAR(45) NULL,
  PRIMARY KEY (`profesor_id`),
  UNIQUE INDEX `profesor_id_UNIQUE` (`profesor_id` ASC) VISIBLE);


CREATE TABLE `prueba_universidad`.`curso` (
  `curso_id` INT NOT NULL AUTO_INCREMENT,
  `curso_nombre` VARCHAR(80) NOT NULL,
  `profesor_id` INT NOT NULL,
  PRIMARY KEY (`curso_id`),
  UNIQUE INDEX `curso_id_UNIQUE` (`curso_id` ASC) VISIBLE);

CREATE TABLE `prueba_universidad`.`calificacion` (
  `calificacion_id` INT NOT NULL AUTO_INCREMENT,
  `estudiante_id` INT NOT NULL,
  `curso_id` INT NOT NULL,
  `calificacion` DECIMAL(3,1) NULL,
  PRIMARY KEY (`calificacion_id`),
  UNIQUE INDEX `calificacion_id_UNIQUE` (`calificacion_id` ASC) VISIBLE);


--Creando las conexiones entre las tablas


ALTER TABLE `prueba_universidad`.`calificacion` 
ADD INDEX `estudiante_id_idx` (`estudiante_id` ASC) VISIBLE,
ADD INDEX `curso_id_idx` (`curso_id` ASC) VISIBLE;
;
ALTER TABLE `prueba_universidad`.`calificacion` 
ADD CONSTRAINT `estudiante_id`
  FOREIGN KEY (`estudiante_id`)
  REFERENCES `prueba_universidad`.`estudiante` (`estudiante_id`)
  ON DELETE CASCADE
  ON UPDATE NO ACTION,
ADD CONSTRAINT `curso_id`
  FOREIGN KEY (`curso_id`)
  REFERENCES `prueba_universidad`.`curso` (`curso_id`)
  ON DELETE CASCADE
  ON UPDATE NO ACTION;

ALTER TABLE `prueba_universidad`.`curso` 
ADD INDEX `profesor_id_idx` (`profesor_id` ASC) VISIBLE;
;
ALTER TABLE `prueba_universidad`.`curso` 
ADD CONSTRAINT `profesor_id`
  FOREIGN KEY (`profesor_id`)
  REFERENCES `prueba_universidad`.`profesor` (`profesor_id`)
  ON DELETE CASCADE
  ON UPDATE NO ACTION;

-- Creando una base de datos de pruebas

INSERT INTO profesor (profesor_nombre, profesor_apellido, departamento)
VALUES
    ('Joaquín', 'Rosa', 'Desarrollo Web'),
    ('Carlos', 'Colina', 'Desarrollo Web');

INSERT INTO estudiante (estudiante_nombre, estudiante_apellido, estudiante_email)
VALUES
    ('Robinson', 'Escalona', 'robinson@example.com'),
    ('Ibzan', 'Capote', 'ibzan@example.com'),
    ('Simon', 'Bolivar', 'simon@example.com'),
    ('Maria', 'Machado', 'maria@example.com'),
    ('Rosa', 'Melano', 'rosa@example.com');

INSERT INTO curso (curso_nombre, profesor_id)
VALUES
    ('CSS', 1),
    ('HTML', 1),
    ('Javascript', 2),
    ('Github', 2),
    ('React', 1);

INSERT INTO calificacion (estudiante_id, curso_id, calificacion)
VALUES
    (1, 1, 85),
    (1, 2, 92),
    (1, 3, 78),
    (1, 4, 95),
    (1, 5, 88),
    (2, 5, 85),
    (2, 4, 92),
    (2, 3, 78),
    (2, 2, 95),
    (3, 1, 88),
    (3, 2, 92),
    (3, 3, 78),
    (4, 4, 95),
    (4, 5, 88),
    (5, 5, 90);

SELECT * FROM estudiante;
SELECT * FROM curso;
SELECT * FROM profesor;
SELECT * FROM calificacion;


-- La nota media que otorga cada profesor 

SELECT 
    p.profesor_nombre, 
    p.profesor_apellido, 
    AVG(c.calificacion) AS promedio_calificaciones
FROM 
    profesor p
INNER JOIN curso cu ON p.profesor_id = cu.profesor_id
INNER JOIN calificacion c ON cu.curso_id = c.curso_id
GROUP BY 
    p.profesor_id, p.profesor_nombre, p.profesor_apellido
ORDER BY 
    promedio_calificaciones DESC;
    
-- Las mejores notas de cada estudiante

SELECT 
    e.estudiante_nombre, 
    e.estudiante_apellido, 
    MAX(c.calificacion) AS mejor_nota
FROM 
    estudiante e
INNER JOIN calificacion c ON e.estudiante_id = c.estudiante_id
GROUP BY 
    e.estudiante_id, e.estudiante_nombre, e.estudiante_apellido;
    
-- Ordenar a los estudiantes por los cursos en los que están inscritos

SELECT 
    e.estudiante_nombre, 
    e.estudiante_apellido, 
    GROUP_CONCAT(cu.curso_nombre SEPARATOR ', ') AS cursos_inscritos
FROM 
    estudiante e
INNER JOIN calificacion c ON e.estudiante_id = c.estudiante_id
INNER JOIN curso cu ON c.curso_id = cu.curso_id
GROUP BY 
    e.estudiante_id, e.estudiante_nombre, e.estudiante_apellido;
    
-- Informe resumido de los cursos y sus calificaciones promedio, ordenados desde el curso más desafiante 
-- (curso con la calificación promedio más baja) hasta el curso más fácil.
    
SELECT 
    cu.curso_nombre, 
    AVG(c.calificacion) AS promedio_calificacion
FROM 
    curso cu
INNER JOIN calificacion c ON cu.curso_id = c.curso_id
GROUP BY 
    cu.curso_id, cu.curso_nombre
ORDER BY 
    promedio_calificacion ASC;
    
-- Encontrar qué estudiante y profesor tienen más cursos en común

WITH CursoEnComun AS (
    SELECT 
        e.estudiante_id, 
        p.profesor_id, 
        COUNT(*) AS cursos_en_comun
    FROM 
        estudiante e
    INNER JOIN calificacion c ON e.estudiante_id = c.estudiante_id
    INNER JOIN curso cu ON c.curso_id = cu.curso_id
    INNER JOIN profesor p ON cu.profesor_id = p.profesor_id
    GROUP BY 
        e.estudiante_id, p.profesor_id
)
SELECT 
    e.estudiante_nombre, 
    e.estudiante_apellido, 
    p.profesor_nombre, 
    p.profesor_apellido, 
    c.cursos_en_comun
FROM 
    CursoEnComun c
INNER JOIN estudiante e ON c.estudiante_id = e.estudiante_id
INNER JOIN profesor p ON c.profesor_id = p.profesor_id
ORDER BY 
    cursos_en_comun DESC
LIMIT 1;
