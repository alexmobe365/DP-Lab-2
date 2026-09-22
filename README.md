# SQL Server — Implement Programmability Objects

## 📌 Descripción

Este repositorio contiene la implementación del laboratorio **"Implement programmability objects with SQL"** de Microsoft Learn, realizado utilizando **SQL Server** y **SQL Server Management Studio (SSMS)**.

El objetivo del laboratorio es trabajar con diferentes objetos de programación de SQL Server para **centralizar lógica, reutilizar consultas y automatizar operaciones sobre los datos**.

Durante el laboratorio se trabajó con la base de datos **AdventureWorksLT**.

## 🛠️ Tecnologías utilizadas

* **SQL Server**
* **SQL Server Management Studio (SSMS)**
* **T-SQL**
* **AdventureWorksLT**
* **Git / GitHub**

## 🗄️ Base de datos

El laboratorio utiliza la base de datos de ejemplo **AdventureWorksLT**, trabajando principalmente con las tablas del esquema `SalesLT`, como:

* `SalesLT.Customer`
* `SalesLT.SalesOrderHeader`
* `SalesLT.SalesOrderDetail`
* `SalesLT.Product`

Antes de comenzar se realizaron consultas para comprobar que las tablas y los datos estaban disponibles correctamente.

## 📋 Desarrollo del laboratorio

### 1. Creación de una View

Se creó la vista:

```sql
SalesLT.vCustomerOrders
```

Esta vista combina información de clientes y pedidos mediante un `INNER JOIN`.

Permite consultar fácilmente:

* ID del cliente.
* Nombre completo.
* ID del pedido.
* Fecha del pedido.

De esta forma, se encapsula la complejidad del `JOIN` y se simplifican las consultas posteriores.

### 2. Creación de un Stored Procedure

Se creó el procedimiento almacenado:

```sql
dbo.AddOrderLineItem
```

Su función es añadir una nueva línea a un pedido existente.

El procedimiento:

1. Recibe `SalesOrderID`, `ProductID` y `Quantity`.
2. Obtiene el precio del producto.
3. Comprueba que el producto exista.
4. Comprueba que el pedido exista.
5. Inserta la nueva línea en `SalesOrderDetail`.
6. Recalcula el subtotal del pedido.
7. Actualiza la fecha de modificación.
8. Utiliza una **transacción** para garantizar que todas las operaciones se realicen correctamente.

En caso de error, la transacción se revierte mediante `ROLLBACK` y se genera una excepción con `THROW`.

Esto permite encapsular una operación de negocio completa dentro de un único procedimiento.

### 3. Creación de una función escalar

Se creó la función:

```sql
dbo.fnOrderTotal
```

Esta función recibe un `OrderID` y devuelve el **importe total del pedido**, calculando la suma de `LineTotal` de sus líneas.

Ejemplo:

```sql
SELECT d.SalesOrderID,
       dbo.fnOrderTotal(d.SalesOrderID) AS OrderTotal
FROM SalesLT.SalesOrderDetail d
GROUP BY d.SalesOrderID;
```

Las funciones escalares permiten reutilizar cálculos en diferentes consultas.

### 4. Creación de una función Table-Valued Function (TVF)

Se creó la función:

```sql
dbo.GetCustomerOrders
```

Esta función recibe un `CustomerID` y devuelve los pedidos asociados a ese cliente.

Al ser una **Inline Table-Valued Function**, puede utilizarse directamente dentro de consultas `SELECT` y combinaciones mediante `JOIN` o `CROSS APPLY`.

Ejemplo:

```sql
SELECT *
FROM dbo.GetCustomerOrders(29929)
ORDER BY OrderDate DESC;
```

También se utilizó `CROSS APPLY` para combinar la función con la tabla de clientes.

### 5. Creación de un sistema de auditoría con Trigger

Se creó una tabla de auditoría:

```sql
dbo.OrderAudit
```

Esta tabla almacena:

* `OrderID`
* `OldTotal`
* `NewTotal`
* `ChangedAt`

Posteriormente se creó el trigger:

```sql
SalesLT.trg_LogOrderTotalChange
```

El trigger se ejecuta automáticamente después de operaciones `INSERT` o `UPDATE` sobre `SalesOrderDetail`.

Su función es detectar los pedidos afectados, calcular sus totales y registrar en `OrderAudit` el valor anterior y el nuevo valor.

### 6. Prueba del Trigger

Para comprobar su funcionamiento se modificó la cantidad de un producto dentro de un pedido:

```sql
UPDATE d
SET OrderQty = OrderQty + 1
FROM SalesLT.SalesOrderDetail d
WHERE d.SalesOrderID =
(
    SELECT TOP 1 SalesOrderID
    FROM SalesLT.SalesOrderHeader
    ORDER BY SalesOrderID DESC
);
```

Posteriormente se consultó:

```sql
SELECT TOP (5) *
FROM dbo.OrderAudit
ORDER BY AuditID DESC;
```

Esto permitió comprobar que el trigger había registrado automáticamente:

* El pedido afectado.
* El total anterior.
* El nuevo total.
* La fecha y hora del cambio.

## 📁 Objetos creados

| Objeto                            | Función                                         |
| --------------------------------- | ----------------------------------------------- |
| `SalesLT.vCustomerOrders`         | Simplifica consultas de clientes y pedidos      |
| `dbo.AddOrderLineItem`            | Añade líneas de pedido y actualiza el subtotal  |
| `dbo.fnOrderTotal`                | Calcula el total de un pedido                   |
| `dbo.GetCustomerOrders`           | Devuelve los pedidos de un cliente              |
| `dbo.OrderAudit`                  | Almacena información de auditoría               |
| `SalesLT.trg_LogOrderTotalChange` | Registra automáticamente cambios en los pedidos |

## 🎯 Conceptos aprendidos

Durante este laboratorio se trabajaron los siguientes conceptos de SQL Server:

* **Views**
* **Stored Procedures**
* **Transactions**
* `BEGIN TRANSACTION`
* `COMMIT`
* `ROLLBACK`
* `THROW`
* **Scalar Functions**
* **Inline Table-Valued Functions (TVF)**
* `CROSS APPLY`
* **Triggers**
* Tablas de auditoría
* Tablas `inserted` y `deleted`
* `CTE`
* Agregaciones con `SUM()`
* Encapsulación de lógica de negocio
* Automatización de operaciones mediante triggers

## ✅ Resultado

Laboratorio completado correctamente, implementando diferentes objetos de **programabilidad de SQL Server** para simplificar consultas, encapsular operaciones de negocio, reutilizar cálculos, crear funciones parametrizadas y registrar automáticamente cambios mediante triggers.

El ejercicio permitió trabajar desde objetos reutilizables como **views y funciones** hasta mecanismos de automatización y auditoría mediante **stored procedures y triggers**.

## 📚 Referencia

Laboratorio realizado siguiendo el ejercicio oficial de Microsoft Learn:

**Implement programmability objects with SQL**

[Microsoft Learn — Implement programmability objects with SQL](https://microsoftlearning.github.io/mslearn-sql-developer/Instructions/Labs/02-implement-programmability-objects.html?utm_source=chatgpt.com)
