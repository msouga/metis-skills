---
name: sql-optimization
description: >
  Guia a Claude en optimizacion de queries SQL Server, diseno de tablas,
  estrategias de indexacion, stored procedures y transacciones T-SQL.
  Usar cuando el usuario escribe o revisa queries T-SQL, disena tablas
  y relaciones, crea o evalua indices, optimiza stored procedures,
  o trabaja con transacciones en bases de datos SQL Server.
---

# Convenciones SQL Server de RSM Peru

## Diseno de tablas

Aplicar estas convenciones en toda tabla nueva:

- Usar `Id INT IDENTITY(1,1) PRIMARY KEY` como clave primaria por defecto.
- Usar `NVARCHAR` para texto (nunca `VARCHAR`) para soportar caracteres latinos.
- Incluir siempre columnas de auditoria:
  - `CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE()`
  - `UpdatedAt DATETIME2 NULL`
- Nombrar foreign keys con prefijo `FK_TablaOrigen_TablaDestino`.
- Nombrar constraints CHECK con prefijo `CK_Tabla_Columna`.
- Crear indice en toda columna de foreign key al momento de crear la tabla.
- Crear indice descendente en `CreatedAt` si la tabla tendra consultas por fecha.
- Usar `ON DELETE SET NULL` en foreign keys opcionales, `ON DELETE CASCADE` solo cuando la entidad hija no tiene sentido sin la padre.

## Nomenclatura de indices

Seguir este patron:

- Indice simple: `IX_Tabla_Columna`
- Indice compuesto: `IX_Tabla_Columna1_Columna2`
- Indice unico: `UX_Tabla_Columna`
- Indice full-text: documentar en comentario antes del `CREATE FULLTEXT INDEX`

Incluir columnas con `INCLUDE` cuando el query frecuente necesita columnas adicionales sin agregarlas al arbol del indice.

## Stored procedures

Seguir esta plantilla para todos los stored procedures:

1. Nombrar con prefijo `sp_` seguido de verbo y entidad: `sp_GetActiveWorkItemsByUser`.
2. Usar `CREATE OR ALTER PROCEDURE`.
3. Incluir `SET NOCOUNT ON` como primera linea del cuerpo.
4. Implementar paginacion con `OFFSET/FETCH` cuando el resultado puede ser grande.
5. Parametros de paginacion por defecto: `@PageSize INT = 50, @PageNumber INT = 1`.
6. Retornar el conteo total como segundo result set para la UI de paginacion.

## Transacciones y auditoria

Toda operacion que modifique mas de una tabla debe:

1. Usar bloque `BEGIN TRY / BEGIN CATCH` con `BEGIN TRANSACTION / COMMIT / ROLLBACK`.
2. Verificar `@@TRANCOUNT > 0` antes de hacer `ROLLBACK` en el bloque CATCH.
3. Insertar registro en la tabla `AuditLog` dentro de la misma transaccion.
4. Estructura de AuditLog: `(EntityType, EntityId, Action, UserId, Timestamp)`.
5. En el CATCH, capturar el error con `ERROR_MESSAGE()` y relanzar con `RAISERROR`.

## Reglas de queries

- Nunca usar `SELECT *`. Listar columnas explicitas.
- Nunca aplicar funciones sobre columnas indexadas en el WHERE (ej: `WHERE YEAR(CreatedAt) = 2025` anula el indice).
- Siempre incluir paginacion en queries que alimentan listas en la UI.
- Usar alias cortos para tablas en JOINs (primera letra o abreviatura).
- Preferir `LEFT JOIN` sobre subconsultas correlacionadas.
- Incluir `ORDER BY` explicito cuando se usa `OFFSET/FETCH`.

## Entity Framework Core

Cuando el proyecto usa EF Core:

- Configurar indices desde `OnModelCreating` con `.HasDatabaseName()` siguiendo la nomenclatura de arriba.
- Usar `HasQueryFilter` para soft-delete (filtrar registros con `Status = 'Deleted'`).
- Llamar stored procedures con `FromSqlInterpolated` (nunca concatenar SQL).
