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
