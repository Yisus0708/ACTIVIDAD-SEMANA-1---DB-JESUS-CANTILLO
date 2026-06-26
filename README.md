# Taller Práctico: Análisis de Requerimientos, Modelo Relacional y Normalización - Riwi Supply

Este repositorio contiene la solución completa al caso práctico de diseño e implementación de una base de datos relacional para la empresa **Riwi Supply**, la cual gestionaba sus operaciones comerciales en una única hoja de cálculo descentralizada.

---

## 📌 Contexto del Caso
La empresa **Riwi Supply** venía registrando la información de sus operaciones comerciales en una única estructura de datos (Excel), almacenando de manera conjunta datos de clientes, sucursales, asesores, productos, categorías, facturas y pagos. El objetivo de este proyecto es migrar dicha información hacia una base de datos relacional para mejorar la integridad, reducir la duplicidad de datos y facilitar la toma de decisiones mediante reportes.

---

## 🚀 Desarrollo del Taller (Punto por Punto)

### 🧩 Fase 1: Comprensión del Negocio

* **¿Qué proceso de negocio representa la información?** Representa el proceso de ventas y distribución de una ferretería/distribuidora de materiales de construcción. Se lleva un registro de los productos ofrecidos y qué empresas los compran con sus respectivas características.
* **¿Qué eventos se están registrando?** Cada vez que un cliente realiza un pedido, se anota en una o varias filas (si el pedido incluye más de un producto).
* **¿Qué actores participan en el proceso?** * **Cliente:** Empresa constructora o ferretería que realiza la compra.
    * **Asesor:** El comercial que gestiona e interviene en la venta.
    * **Sucursal:** La sede física desde donde se despacha la mercancía.
* **¿Qué información parece repetirse?** Se duplica la información de los clientes (ej. *Constructora Andina SAS* aparece 3 veces con la misma ciudad y vendedor) y de los asesores (ej. *Carlos Pérez* u *Andrés Ruiz* se repiten con cada venta que atienden). Esto consume espacio innecesario y genera riesgos de inconsistencia.
* **¿Qué problemas podrían presentarse si la empresa continúa almacenando los datos de esta manera?** * **Datos inconsistentes:** Ciudades o nombres escritos de forma diferente para una misma entidad en distintas facturas.
    * **Datos duplicados sin control:** Redundancia masiva de datos fijos de vendedores y clientes.
    * **Pérdida de historial (Anomalías de borrado):** Si se borra la fila de un producto único en una factura específica (ej. *Taladro Industrial* en `FAC-2002`), la existencia de dicha venta o la traza del producto desaparece del sistema sin advertencia.

---

### 🗂️ Fase 2: Identificación de Entidades

A partir del análisis, se abstrajeron las siguientes entidades del mundo real del negocio:

1. **cliente:** Empresa u organización que realiza la compra de productos a Riwi Supply.
2. **vendedor:** Asesor comercial que gestiona, atiende y hace el seguimiento de la venta.
3. **sucursal:** Sede física de Riwi Supply desde donde se realiza la operación logística y el despacho.
4. **categoria:** Clasificación o agrupación a la que pertenece un producto determinado.
5. **producto:** Artículo o material de construcción ofrecido por Riwi Supply para la venta.
6. **metodo_pago:** Forma o medio en que el cliente cancela la transacción financiera.
7. **factura:** Documento de cabecera que registra la transacción general, fecha y actores involucrados.
8. **detalle_factura:** Línea de transacción que rompe la relación de muchos a muchos, registrando cada producto individual y su cantidad dentro de una factura.

---

### 📊 Fase 3: Identificación de Atributos (Diccionario de Datos)

A continuación, se detalla la estructura lógica inicial de las tablas, definiendo sus Claves Primarias (**PK**) y Claves Foráneas (**FK**):

| Entidad | Atributo | Tipo de Dato | PK | FK | Descripción / Restricción |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **categoria** | id_categoria | integer | ✔ | | Identificador único de la categoría. |
| | nombre | varchar(50) | | | Nombre de la categoría (Not Null). |
| **cliente** | id_cliente | integer | ✔ | | Identificador único del cliente. |
| | nombre | varchar(100) | | | Nombre o Razón Social (Not Null). |
| | ciudad | varchar(50) | | | Ciudad base del cliente (Not Null). |
| **vendedor** | id_vendedor | integer | ✔ | | Identificador único del asesor comercial. |
| | nombre | varchar(100) | | | Nombre completo del asesor (Not Null). |
| **sucursal** | id_sucursal | integer | ✔ | | Identificador de la sede de Riwi Supply. |
| | nombre | varchar(50) | | | Nombre de la sucursal (Not Null). |
| | ciudad | varchar(50) | | | Ciudad donde está ubicada (Not Null). |
| **metodo_pago** | id_metodo | integer | ✔ | | Identificador del medio de pago. |
| | nombre | varchar(30) | | | Ejemplo: Efectivo, Transferencia (Not Null). |
| **producto** | id_producto | integer | ✔ | | Identificador único del artículo. |
| | nombre | varchar(100) | | | Descripción comercial del artículo (Not Null). |
| | precio_unitario| decimal | | | Valor unitario base (Not Null). |
| | id_categoria | integer | | ✔ | Relación con la tabla categoria (Not Null). |
| **factura** | id_factura | varchar(10) | ✔ | | Número único de factura (Not Null). |
| | fecha | date | | | Fecha de emisión (Not Null). |
| | id_metodo | integer | | ✔ | Relación con el medio de pago (Not Null). |
| | id_cliente | integer | | ✔ | Relación con el comprador (Not Null). |
| | id_vendedor | integer | | ✔ | Relación con el asesor (Not Null). |
| | id_sucursal | integer | | ✔ | Relación con la sucursal de despacho (Not Null). |
| **detalle_factura**| id_detalle | integer | ✔ | | Identificador autoincremental de la línea. |
| | cantidad | integer | | | Cantidad de artículos comprados (Not Null). |
| | id_factura | varchar(10) | | ✔ | Vinculación a la cabecera de la factura. |
| | id_producto | integer | | ✔ | Vinculación al producto vendido. |

---

### 🗺️ Fase 4 y 5: Identificación de Relaciones y Refinamiento

#### Preguntas de Control y Validation:
* **¿Existen atributos duplicados?** No, cada atributo atómico ha sido asignado de manera estricta a una sola tabla según su naturaleza.
* **¿Existen dependencias parciales?** No. En `detalle_factura`, la métrica `cantidad` requiere de manera obligatoria la combinación lógica del producto y la factura para tener sentido comercial.
* **¿Existen dependencias transitivas?** Sí, originalmente existían en el set de datos (ej. la ciudad del cliente dependía del nombre del cliente y no de la factura). Estas dependencias fueron totalmente eliminadas al segmentar el diseño en tablas maestras e independientes.
* **¿Las relaciones reflejan la realidad del negocio?** Sí. El modelo permite, por ejemplo, que un cliente localizado en Cartagena sea atendido perfectamente de manera excepcional por la sucursal de Barranquilla (como ocurre en la transacción real `FAC-2009`).

#### 📐 Diagrama Entidad-Relación Preliminar y Refinado
<img width="1536" height="1024" alt="diagrama" src="https://github.com/user-attachments/assets/8c25c667-3acb-467c-9f3b-29a2924af75b" />

---

### 📐 Fase 6: Proceso de Normalización

El diseño se sometió a las tres reglas de normalización de bases de datos relacionales:

1. **Primera Forma Normal (1FN) - Atomicidad:** El Excel original carecía de llaves primarias explícitas y repetía la estructura de facturas (como `FAC-2001`) ocupando múltiples renglones para diferentes productos. Se eliminaron los grupos repetitivos extrayendo las líneas de compra hacia la nueva entidad agregada `detalle_factura`, y estableciendo identificadores únicos unívocos.
2. **Segunda Forma Normal (2FN) - Dependencias Parciales:** Al aislar las llaves complejas, se identificó que atributos como la fecha de orden y los datos del cliente dependían únicamente del número global de factura, mientras que el precio unitario dependía directo del artículo. Se procedió a romper la estructura en las tablas independientes `factura` y `producto`.
3. **Tercera Forma Normal (3FN) - Dependencias Transitivas:** Se detectó que datos secundarios como la ciudad de la constructora dependían del cliente, la categoría dependía del producto y la sucursal/método de pago se reiteraban textualmente de forma horizontal. Se normalizó creando de raíz las tablas maestras: `cliente`, `sucursal`, `categoria` y `metodo_pago`.

---

### 💻 Fase 7 y 8: Modelo Relacional Final e Implementación

Aquí se muestra la representación lógica final de la base de datos lista para producción junto con su respectivo Script DDL y DML:

<img width="1896" height="1357" alt="Untitled" src="https://github.com/user-attachments/assets/d40c13a1-1524-4219-aabd-89918a198848" />

```sql
-- crear tablas

create table categoria (
    id_categoria serial primary key,
    nombre varchar(50) not null
);

create table cliente (
    id_cliente serial primary key,
    nombre varchar(100) not null,
    ciudad varchar(50) not null
);

create table vendedor (
    id_vendedor serial primary key,
    nombre varchar(100) not null
);

create table sucursal (
    id_sucursal serial primary key,
    nombre varchar(50) not null,
    ciudad varchar(50) not null
);

create table metodo_pago (
    id_metodo serial primary key,
    nombre varchar(30) not null
);

create table producto (
    id_producto serial primary key,
    nombre varchar(100) not null,
    precio_unitario decimal not null,
    id_categoria integer not null,
    foreign key (id_categoria) references categoria(id_categoria)
);

create table factura (
    id_factura varchar(10) primary key,
    fecha date not null,
    id_metodo integer not null,
    id_cliente integer not null,
    id_vendedor integer not null,
    id_sucursal integer not null,
    foreign key (id_metodo) references metodo_pago(id_metodo),
    foreign key (id_cliente) references cliente(id_cliente),
    foreign key (id_vendedor) references vendedor(id_vendedor),
    foreign key (id_sucursal) references sucursal(id_sucursal)
);

create table detalle_factura (
    id_detalle serial primary key,
    cantidad integer not null,
    id_factura varchar(10) not null,
    id_producto integer not null,
    foreign key (id_factura) references factura(id_factura),
    foreign key (id_producto) references producto(id_producto)
);

-- insertar datos

insert into categoria (nombre) values
('construcción'),
('herramientas'),
('pinturas'),
('agregados');

insert into cliente (nombre, ciudad) values
('constructora andina sas', 'bogotá'),
('ferretería el tornillo', 'medellín'),
('inversiones caribe sas', 'barranquilla'),
('metalúrgica del norte', 'cali'),
('obras y vías sas', 'bucaramanga'),
('constructora pacífico', 'cali'),
('ferretería central', 'cartagena');

insert into vendedor (nombre) values
('carlos pérez'),
('laura gómez'),
('andrés ruiz'),
('diana torres'),
('miguel castro');

insert into sucursal (nombre, ciudad) values
('centro', 'bogotá'),
('norte', 'medellín'),
('costa', 'barranquilla'),
('occidente', 'cali'),
('oriente', 'bucaramanga');

insert into metodo_pago (nombre) values
('transferencia'),
('tarjeta'),
('crédito'),
('efectivo');

insert into producto (nombre, precio_unitario, id_categoria) values
('cemento gris 50kg', 32000, 1),
('varilla 3/8', 28000, 1),
('taladro industrial', 450000, 2),
('pintura blanca 5gl', 95000, 3),
('rodillo profesional', 18000, 3),
('disco de corte', 12000, 2),
('broca sds', 15000, 2),
('arena lavada', 25000, 4),
('grava triturada', 27000, 4);

insert into factura (id_factura, fecha, id_metodo, id_cliente, id_vendedor, id_sucursal) values
('fac-2001', '2026-05-01', 1, 1, 1, 1),
('fac-2002', '2026-05-02', 2, 2, 2, 2),
('fac-2003', '2026-05-03', 3, 3, 3, 3),
('fac-2004', '2026-05-04', 4, 4, 4, 4),
('fac-2005', '2026-05-05', 1, 1, 1, 1),
('fac-2006', '2026-05-06', 2, 2, 2, 2),
('fac-2007', '2026-05-07', 3, 5, 5, 5),
('fac-2008', '2026-05-08', 1, 6, 4, 4),
('fac-2009', '2026-05-09', 2, 7, 3, 3);

insert into detalle_factura (cantidad, id_factura, id_producto) values
(100, 'fac-2001', 1),
(50, 'fac-2001', 2),
(3, 'fac-2002', 3),
(20, 'fac-2003', 4),
(10, 'fac-2003', 5),
(40, 'fac-2004', 6),
(120, 'fac-2005', 1),
(25, 'fac-2006', 7),
(60, 'fac-2007', 8),
(80, 'fac-2007', 9),
(15, 'fac-2008', 4),
(2, 'fac-2009', 3);
```
---

### 🔍 Fase 9: Consultas SQL de Reportes Solicitadas

A continuación, se listan los requerimientos de extracción analítica resueltos para la gerencia de Riwi Supply:

1.  **Listar todos los clientes:**
    ```sql
    SELECT * FROM cliente;
    ```
2.  **Mostrar todos los productos con su categoría:**
    ```sql
    SELECT p.nombre AS Producto, c.nombre AS Categoria 
    FROM producto p
    INNER JOIN categoria c ON p.id_categoria = c.id_categoria;
    ```
3.  **Consultar las facturas emitidas por ciudad (de la sucursal):**
    ```sql
    SELECT s.ciudad, COUNT(f.id_factura) AS Total_Facturas
    FROM factura f
    INNER JOIN sucursal s ON f.id_sucursal = s.id_sucursal
    GROUP BY s.ciudad;
    ```
4.  **Obtener el total vendido por cliente:**
    ```sql
    SELECT c.nombre AS Cliente, SUM(df.cantidad * p.precio_unitario) AS Total_Comprado
    FROM detalle_factura df
    INNER JOIN factura f ON df.id_factura = f.id_factura
    INNER JOIN cliente c ON f.id_cliente = c.id_cliente
    INNER JOIN producto p ON df.id_producto = p.id_producto
    GROUP BY c.nombre;
    ```
5.  **Obtener el total vendido por categoría:**
    ```sql
    SELECT cat.nombre AS Categoria, SUM(df.cantidad * p.precio_unitario) AS Total_Vendido
    FROM detalle_factura df
    INNER JOIN producto p ON df.id_producto = p.id_producto
    INNER JOIN categoria cat ON p.id_categoria = cat.id_categoria
    GROUP BY cat.nombre;
    ```
6.  **Mostrar las facturas atendidas por cada asesor comercial:**
    ```sql
    SELECT v.nombre AS Asesor, f.id_factura, f.fecha
    FROM factura f
    INNER JOIN vendedor v ON f.id_vendedor = v.id_vendedor;
    ```
7.  **Consultar los productos más vendidos:**
    ```sql
    SELECT p.nombre AS Producto, SUM(df.cantidad) AS Unidades_Vendidas
    FROM detalle_factura df
    INNER JOIN producto p ON df.id_producto = p.id_producto
    GROUP BY p.nombre
    ORDER BY Unidades_Vendidas DESC;
    ```
8.  **Mostrar las sucursales y la cantidad de facturas gestionadas:**
    ```sql
    SELECT s.nombre AS Sucursal, COUNT(f.id_factura) AS Cantidad_Facturas
    FROM factura f
    INNER JOIN sucursal s ON f.id_sucursal = s.id_sucursal
    GROUP BY s.nombre;
    ```
9.  **Consultar ventas realizadas mediante un método de pago específico (Ej: 'Efectivo'):**
    ```sql
    SELECT f.id_factura, f.fecha, mp.nombre AS Metodo_Pago
    FROM factura f
    INNER JOIN metodo_pago mp ON f.id_metodo = mp.id_metodo
    WHERE mp.nombre = 'Efectivo';
    ```
10. **Obtener el valor total de cada factura:**
    ```sql
    SELECT id_factura, SUM(cantidad * precio_unitario) AS Valor_Total_Factura
    FROM detalle_factura df
    INNER JOIN producto p ON df.id_producto = p.id_producto
    GROUP BY id_factura;
    ```

---

## 📝 Conclusiones y Lecciones Aprendidas
* **Integridad de Datos:** La transición de datos planos a un ecosistema relacional previene errores de actualización humana y elimina de forma absoluta las redundancias no deseadas.
* **Normalización Eficiente:** Aplicar correctamente las reglas de normalización (1FN, 2FN, 3FN) garantiza que el rendimiento de las consultas y la escalabilidad del negocio no se degraden con el paso del tiempo.
