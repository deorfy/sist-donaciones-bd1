# sist-donaciones-bd1
# Sistema de Gestión de Donaciones - Casas de Acogida (Santa Cruz, Bolivia)

Este repositorio contiene el diseño lógico, la implementación del esquema físico (DDL), la inserción de datos de prueba (DML), consultas avanzadas y vistas del **Sistema de Gestión de Donaciones para Casas de Acogida** en el departamento de Santa Cruz, Bolivia.

El sistema permite centralizar los requerimientos críticos de diferentes hogares de acogida (víveres, medicamentos, infraestructura, etc.) y conectar de manera eficiente a los donantes (particulares u organizaciones) con dichas necesidades.

## 📋 Información del Proyecto
* **Materia:** Base de Datos I (INF-312)
* **Estudiante:** Delgado Teran Kalet Manre
* **Fecha:** 30 de junio de 2026
* **Entorno de BD:** SQLite

---
```sql
-- ============================================================
-- DDL: CREAR TABLAS
-- ============================================================

CREATE TABLE MUNICIPIO (
    Id_Municipio INTEGER PRIMARY KEY AUTOINCREMENT,
    Nombre       VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE USUARIO (
    Id_User        INTEGER PRIMARY KEY AUTOINCREMENT,
    Nombre         VARCHAR(100) NOT NULL,
    Apellido       VARCHAR(100) NOT NULL,
    Email          VARCHAR(150) NOT NULL UNIQUE,
    Contraseña     VARCHAR(255) NOT NULL,
    Rol            VARCHAR(20)  NOT NULL CHECK (Rol IN ('Administrador', 'Donante', 'Beneficiario')),
    Genero         VARCHAR(20),
    Fecha_Registro DATE         NOT NULL DEFAULT (DATE('now'))
);

CREATE TABLE DONANTE (
    Id_User             INTEGER PRIMARY KEY,
    Nombre_Organizacion VARCHAR(150),
    Es_Anonimo          BOOLEAN NOT NULL DEFAULT 0,
    FOREIGN KEY (Id_User) REFERENCES USUARIO(Id_User)
        ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE BENEFICIARIO (
    Id_User             INTEGER PRIMARY KEY,
    Nombre_Organizacion VARCHAR(150) NOT NULL,
    Eslogan_Corto       VARCHAR(200),
    Ruta_Imagen_Portada VARCHAR(300),
    Id_Municipio        INTEGER NOT NULL,
    Direccion           VARCHAR(200),
    Horarios            VARCHAR(150),
    Presentacion_Hogar  TEXT,
    Ruta_Qr_Tarjeta     VARCHAR(300),
    FOREIGN KEY (Id_User) REFERENCES USUARIO(Id_User)
        ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (Id_Municipio) REFERENCES MUNICIPIO(Id_Municipio)
        ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE CAT_ELEMENTO (
    Id_CatElemento INTEGER PRIMARY KEY AUTOINCREMENT,
    Tipo           VARCHAR(50) NOT NULL CHECK (Tipo IN ('Dinero', 'Víveres', 'Ropa', 'Medicamentos', 'Otro')),
    Descripcion    VARCHAR(200),
    Nombre_Icono   VARCHAR(100)
);

CREATE TABLE NECESIDAD_HOGAR (
    Id_User                INTEGER NOT NULL,
    Id_CatElemento         INTEGER NOT NULL,
    Descripcion_Especifica VARCHAR(300),
    Estado                 VARCHAR(20) NOT NULL DEFAULT 'Moderado'
                           CHECK (Estado IN ('Satisfecho', 'Moderado', 'Crítico')),
    Ultima_Actualizacion   DATE NOT NULL DEFAULT (DATE('now')),
    PRIMARY KEY (Id_User, Id_CatElemento),
    FOREIGN KEY (Id_User) REFERENCES BENEFICIARIO(Id_User)
        ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (Id_CatElemento) REFERENCES CAT_ELEMENTO(Id_CatElemento)
        ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE DONACION (
    Id_Donacion          INTEGER PRIMARY KEY AUTOINCREMENT,
    Id_Donante           INTEGER NOT NULL,
    Id_User_Beneficiario INTEGER NOT NULL,
    Id_CatElemento       INTEGER NOT NULL,
    Fecha_Donacion       DATE    NOT NULL DEFAULT (DATE('now')),
    Monto_O_Cantidad     DECIMAL(10,2),
    Comentario_Donante   TEXT,
    FOREIGN KEY (Id_Donante) REFERENCES DONANTE(Id_User)
        ON DELETE RESTRICT ON UPDATE CASCADE,
    FOREIGN KEY (Id_User_Beneficiario, Id_CatElemento)
        REFERENCES NECESIDAD_HOGAR(Id_User, Id_CatElemento)
        ON DELETE RESTRICT ON UPDATE CASCADE
);
-- ============================================================
-- ÍNDICES
-- ============================================================

CREATE INDEX idx_usuario_email    ON USUARIO(Email);
CREATE INDEX idx_usuario_rol      ON USUARIO(Rol);
CREATE INDEX idx_donacion_donante ON DONACION(Id_Donante);
CREATE INDEX idx_donacion_fecha   ON DONACION(Fecha_Donacion);
CREATE INDEX idx_necesidad_estado ON NECESIDAD_HOGAR(Estado);
-- ============================================================
-- DML: DATOS DE PRUEBA
-- ============================================================

INSERT INTO MUNICIPIO (Nombre) VALUES
('Santa Cruz de la Sierra'),
('Warnes'),
('Montero'),
('Cotoca'),
('La Guardia'),
('El Torno'),
('Portachuelo'),
('Camiri'),
('Yapacaní'),
('San Ignacio de Velasco');

INSERT INTO USUARIO (Nombre, Apellido, Email, Contraseña, Rol, Genero, Fecha_Registro) VALUES
('Carlos',   'Mendoza',   'admin@donaciones.bo',      'hash1', 'Administrador', 'Masculino',  '2024-01-01'),
('Maria',    'Flores',    'maria.flores@gmail.com',   'hash2', 'Donante',       'Femenino',   '2024-02-10'),
('Jorge',    'Suarez',    'jorge.suarez@gmail.com',   'hash3', 'Donante',       'Masculino',  '2024-03-05'),
('Lucia',    'Perez',     'lucia.perez@gmail.com',    'hash4', 'Donante',       'Femenino',   '2024-03-15'),
('Roberto',  'Vaca',      'roberto.vaca@gmail.com',   'hash5', 'Donante',       'Masculino',  '2024-04-01'),
('Ana',      'Gutierrez', 'ana.gutierrez@gmail.com',  'hash6', 'Donante',       'Femenino',   '2024-04-20'),
('Hogar',    'SanJose',   'hogar.sanjose@gmail.com',  'hash7', 'Beneficiario',  NULL,         '2024-01-15'),
('Hogar',    'LuzEsper',  'hogar.luzesper@gmail.com', 'hash8', 'Beneficiario',  NULL,         '2024-01-20'),
('Hogar',    'NinosDios', 'hogar.ninosdios@gmail.com','hash9', 'Beneficiario',  NULL,         '2024-02-01'),
('Hogar',    'AncianosF', 'hogar.ancianos@gmail.com', 'hash10','Beneficiario',  NULL,         '2024-02-10');

INSERT INTO DONANTE (Id_User, Nombre_Organizacion, Es_Anonimo) VALUES
(2, NULL,                    0),
(3, 'Empresa Suarez & Hnos', 0),
(4, NULL,                    1),
(5, 'Constructora Vaca',     0),
(6, NULL,                    0);

INSERT INTO BENEFICIARIO (Id_User, Nombre_Organizacion, Eslogan_Corto, Ruta_Imagen_Portada, Id_Municipio, Direccion, Horarios, Presentacion_Hogar, Ruta_Qr_Tarjeta) VALUES
(7,  'Hogar San Lorenzo',         'Un hogar para cada niño',        '/img/sanlorenzo.jpg', 1, 'Av. Banzer Km 5',      '8:00 - 16:00', 'Hogar de acogida para niños de 0 a 12 años en situación de vulnerabilidad.', '/qr/sanjose.png'),
(8,  'Hogar Fatima',             'Iluminando vidas con amor',      '/img/fatima.jpg',     1, 'Calle Sucre #234',     '7:00 - 13:00', 'Refugio para niñas y adolescentes en riesgo social.',                         '/qr/luzesper.png'),
(9,  'Hogar Madre Mia',          'Fe, amor y esperanza',           '/img/ninos.jpg',      4, 'Barrio Los Lotes #12', '12:00 - 17:00', 'Hogar católico para niños huérfanos y en abandono.',                          '/qr/ninos.png'),
(10, 'Hogar Ancianos Felices',   'Dignidad en cada etapa de vida', '/img/ancianos.jpg',   2, 'Av. Principal #89',    '6:00 - 11:00', 'Casa de reposo y acogida para adultos mayores sin familia.',                   '/qr/ancianos.png');

INSERT INTO CAT_ELEMENTO (Tipo, Descripcion, Nombre_Icono) VALUES
('Dinero',       'Aporte monetario directo al hogar',           'icon-money.png'),
('Víveres',      'Alimentos no perecederos y perecederos',      'icon-food.png'),
('Ropa',         'Prendas de vestir para niños y adultos',      'icon-clothes.png'),
('Medicamentos', 'Medicinas y suplementos vitamínicos',         'icon-medicine.png'),
('Otro',         'Materiales escolares, juguetes, muebles etc', 'icon-other.png');

INSERT INTO NECESIDAD_HOGAR (Id_User, Id_CatElemento, Descripcion_Especifica, Estado, Ultima_Actualizacion) VALUES
(7,  1, 'Necesitamos fondos para pagar servicios básicos',         'Crítico',    '2024-06-01'),
(7,  2, 'Arroz, fideo, aceite y leche para 20 niños',             'Moderado',   '2024-06-05'),
(7,  3, 'Ropa de invierno tallas 4 a 12',                         'Crítico',    '2024-06-10'),
(8,  1, 'Apoyo económico para alquiler del local',                'Crítico',    '2024-06-01'),
(8,  4, 'Vitaminas y antigripales para las niñas',                'Moderado',   '2024-06-08'),
(9,  2, 'Alimentos para 15 niños durante el mes',                 'Moderado',   '2024-06-03'),
(9,  5, 'Útiles escolares para inicio de clases',                 'Satisfecho', '2024-06-12'),
(10, 1, 'Fondos para medicación mensual de adultos mayores',      'Crítico',    '2024-06-01'),
(10, 4, 'Medicamentos para hipertensión y diabetes',              'Crítico',    '2024-06-07'),
(10, 2, 'Alimentos blandos y nutritivos para adultos mayores',    'Moderado',   '2024-06-09');

INSERT INTO DONACION (Id_Donante, Id_User_Beneficiario, Id_CatElemento, Fecha_Donacion, Monto_O_Cantidad, Comentario_Donante) VALUES
(2,  7,  1, '2024-06-02', 500.00,  'Con mucho cariño para los niños'),
(3,  7,  2, '2024-06-03', 150.00,  'Donación de víveres de nuestra empresa'),
(4,  8,  1, '2024-06-04', 300.00,  NULL),
(5,  10, 1, '2024-06-05', 1000.00, 'Apoyo para los abuelitos'),
(6,  8,  4, '2024-06-06', 200.00,  'Medicamentos para las niñas'),
(2,  9,  2, '2024-06-07', 100.00,  'Alimentos para los pequeños'),
(3,  10, 4, '2024-06-08', 450.00,  'Medicamentos donados por Suarez & Hnos'),
(5,  7,  3, '2024-06-09', 80.00,   'Ropa de invierno para los niños'),
(6,  9,  5, '2024-06-10', 60.00,   'Útiles escolares'),
(2,  10, 2, '2024-06-11', 120.00,  'Alimentos para los adultos mayores');

-- ============================================================
-- CONSULTAS REQUERIDAS
-- ============================================================

-- 1. Listar todos los hogares beneficiarios con su municipio
SELECT b.Nombre_Organizacion, m.Nombre AS Municipio, b.Direccion
FROM BENEFICIARIO b
JOIN MUNICIPIO m ON b.Id_Municipio = m.Id_Municipio
ORDER BY b.Nombre_Organizacion;

-- 2. Mostrar las necesidades actualmente en estado "Crítico"
SELECT b.Nombre_Organizacion, ce.Tipo, nh.Descripcion_Especifica, nh.Ultima_Actualizacion
FROM NECESIDAD_HOGAR nh
JOIN BENEFICIARIO b ON nh.Id_User = b.Id_User
JOIN CAT_ELEMENTO ce ON nh.Id_CatElemento = ce.Id_CatElemento
WHERE nh.Estado = 'Crítico';

-- 3. Contar las donaciones realizadas por cada donante
SELECT u.Nombre, u.Apellido, COUNT(d.Id_Donacion) AS Total_Donaciones
FROM DONANTE dn
JOIN USUARIO u ON dn.Id_User = u.Id_User
LEFT JOIN DONACION d ON dn.Id_User = d.Id_Donante
GROUP BY dn.Id_User
ORDER BY Total_Donaciones DESC;

-- 4. Listar los hogares con más necesidades en estado crítico
SELECT b.Nombre_Organizacion, COUNT(*) AS Necesidades_Criticas
FROM NECESIDAD_HOGAR nh
JOIN BENEFICIARIO b ON nh.Id_User = b.Id_User
WHERE nh.Estado = 'Crítico'
GROUP BY b.Id_User
ORDER BY Necesidades_Criticas DESC;

-- 5. Mostrar el historial completo de donaciones de un donante (por email)
SELECT d.Fecha_Donacion, b.Nombre_Organizacion, ce.Tipo, d.Monto_O_Cantidad, d.Comentario_Donante
FROM DONACION d
JOIN DONANTE dn ON d.Id_Donante = dn.Id_User
JOIN USUARIO u ON dn.Id_User = u.Id_User
JOIN BENEFICIARIO b ON d.Id_User_Beneficiario = b.Id_User
JOIN CAT_ELEMENTO ce ON d.Id_CatElemento = ce.Id_CatElemento
WHERE u.Email = 'maria.flores@gmail.com'
ORDER BY d.Fecha_Donacion DESC;

-- 6. Calcular el monto total donado por categoría
SELECT ce.Tipo, SUM(d.Monto_O_Cantidad) AS Total_Donado
FROM DONACION d
JOIN CAT_ELEMENTO ce ON d.Id_CatElemento = ce.Id_CatElemento
GROUP BY ce.Tipo
ORDER BY Total_Donado DESC;

-- 7. Listar los 5 donantes que más han aportado en dinero
SELECT u.Nombre, u.Apellido, SUM(d.Monto_O_Cantidad) AS Total_Aportado
FROM DONACION d
JOIN DONANTE dn ON d.Id_Donante = dn.Id_User
JOIN USUARIO u ON dn.Id_User = u.Id_User
WHERE d.Id_CatElemento = (SELECT Id_CatElemento FROM CAT_ELEMENTO WHERE Tipo = 'Dinero')
GROUP BY dn.Id_User
ORDER BY Total_Aportado DESC
LIMIT 5;

-- 8. Mostrar hogares que NO han recibido donaciones en los últimos 30 días
SELECT b.Nombre_Organizacion
FROM BENEFICIARIO b
WHERE b.Id_User NOT IN (
    SELECT Id_User_Beneficiario FROM DONACION
    WHERE Fecha_Donacion >= DATE('now', '-30 days')
);

-- 9. Calcular el total donado por cada hogar beneficiario
SELECT b.Nombre_Organizacion, SUM(d.Monto_O_Cantidad) AS Total_Recibido
FROM DONACION d
JOIN BENEFICIARIO b ON d.Id_User_Beneficiario = b.Id_User
GROUP BY b.Id_User
ORDER BY Total_Recibido DESC;

-- 10. Listar donantes anónimos y su número de donaciones
SELECT dn.Id_User, COUNT(d.Id_Donacion) AS Num_Donaciones
FROM DONANTE dn
LEFT JOIN DONACION d ON dn.Id_User = d.Id_Donante
WHERE dn.Es_Anonimo = 1
GROUP BY dn.Id_User;

-- 11. Mostrar la cantidad de necesidades por estado, agrupadas por hogar
SELECT b.Nombre_Organizacion, nh.Estado, COUNT(*) AS Cantidad
FROM NECESIDAD_HOGAR nh
JOIN BENEFICIARIO b ON nh.Id_User = b.Id_User
GROUP BY b.Id_User, nh.Estado
ORDER BY b.Nombre_Organizacion;

-- 12. Listar todas las donaciones realizadas hoy
SELECT u.Nombre, u.Apellido, b.Nombre_Organizacion, d.Monto_O_Cantidad
FROM DONACION d
JOIN DONANTE dn ON d.Id_Donante = dn.Id_User
JOIN USUARIO u ON dn.Id_User = u.Id_User
JOIN BENEFICIARIO b ON d.Id_User_Beneficiario = b.Id_User
WHERE d.Fecha_Donacion = DATE('now');

-- 13. Encontrar hogares en el mismo municipio que necesitan el mismo tipo de elemento
SELECT b1.Nombre_Organizacion AS Hogar1, b2.Nombre_Organizacion AS Hogar2, ce.Tipo
FROM NECESIDAD_HOGAR nh1
JOIN NECESIDAD_HOGAR nh2 ON nh1.Id_CatElemento = nh2.Id_CatElemento AND nh1.Id_User < nh2.Id_User
JOIN BENEFICIARIO b1 ON nh1.Id_User = b1.Id_User
JOIN BENEFICIARIO b2 ON nh2.Id_User = b2.Id_User
JOIN CAT_ELEMENTO ce ON nh1.Id_CatElemento = ce.Id_CatElemento
WHERE b1.Id_Municipio = b2.Id_Municipio;

-- 14. Calcular el promedio donado por categoría
SELECT ce.Tipo, AVG(d.Monto_O_Cantidad) AS Promedio_Donado
FROM DONACION d
JOIN CAT_ELEMENTO ce ON d.Id_CatElemento = ce.Id_CatElemento
GROUP BY ce.Tipo;

-- 15. Listar hogares por municipio con totales de necesidades activas
SELECT m.Nombre AS Municipio, COUNT(nh.Id_User) AS Necesidades_Activas
FROM NECESIDAD_HOGAR nh
JOIN BENEFICIARIO b ON nh.Id_User = b.Id_User
JOIN MUNICIPIO m ON b.Id_Municipio = m.Id_Municipio
WHERE nh.Estado != 'Satisfecho'
GROUP BY m.Id_Municipio
ORDER BY Necesidades_Activas DESC;

-- ============================================================
-- VISTAS
-- ============================================================

-- Vista 1: Necesidades críticas activas con datos del hogar
CREATE VIEW V_NECESIDADES_CRITICAS AS
SELECT 
    b.Nombre_Organizacion AS Hogar,
    m.Nombre AS Municipio,
    ce.Tipo AS Categoria,
    nh.Descripcion_Especifica,
    nh.Ultima_Actualizacion
FROM NECESIDAD_HOGAR nh
JOIN BENEFICIARIO b ON nh.Id_User = b.Id_User
JOIN MUNICIPIO m ON b.Id_Municipio = m.Id_Municipio
JOIN CAT_ELEMENTO ce ON nh.Id_CatElemento = ce.Id_CatElemento
WHERE nh.Estado = 'Crítico';

-- Vista 2: Historial resumido de donaciones por donante
CREATE VIEW V_HISTORIAL_DONANTE AS
SELECT 
    u.Id_User,
    u.Nombre,
    u.Apellido,
    dn.Es_Anonimo,
    COUNT(d.Id_Donacion) AS Total_Donaciones,
    SUM(d.Monto_O_Cantidad) AS Monto_Total_Aportado
FROM DONANTE dn
JOIN USUARIO u ON dn.Id_User = u.Id_User
LEFT JOIN DONACION d ON dn.Id_User = d.Id_Donante
GROUP BY dn.Id_User;

-- Vista 3: Resumen de donaciones recibidas por hogar
CREATE VIEW V_RESUMEN_HOGAR AS
SELECT 
    b.Id_User,
    b.Nombre_Organizacion,
    m.Nombre AS Municipio,
    COUNT(d.Id_Donacion) AS Total_Donaciones_Recibidas,
    SUM(d.Monto_O_Cantidad) AS Monto_Total_Recibido
FROM BENEFICIARIO b
JOIN MUNICIPIO m ON b.Id_Municipio = m.Id_Municipio
LEFT JOIN DONACION d ON b.Id_User = d.Id_User_Beneficiario
GROUP BY b.Id_User;

