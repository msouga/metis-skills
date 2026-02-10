---
name: ef-core-patterns
description: Aplica las convenciones de Entity Framework Core de RSM Peru para proyectos .NET. Usar cuando se configure un DbContext, se creen entidades con auditoria, se escriban configuraciones Fluent API, o se ejecuten migraciones en soluciones multi-proyecto (API + Infrastructure).
---

# Convenciones EF Core de RSM Peru

## Estructura de solucion

Los proyectos de RSM Peru usan Clean Architecture con dos proyectos relevantes para EF Core:

- `Infrastructure`: contiene el DbContext, configuraciones de entidades y migraciones.
- `API` (o `Web`): es el startup project que tiene el connection string.

Todos los comandos de migracion requieren ambos flags:

```bash
dotnet ef migrations add NombreDescriptivo \
    --project Infrastructure \
    --startup-project API

dotnet ef database update \
    --project Infrastructure \
    --startup-project API
```

Para generar scripts SQL idempotentes (requerido para despliegues en Azure):

```bash
dotnet ef migrations script \
    --project Infrastructure \
    --startup-project API \
    --idempotent \
    --output migration.sql
```

## Interfaz IAuditable

Todas las entidades de negocio deben implementar `IAuditable`. La interfaz define:

- `CreatedAt` (DateTime, UTC)
- `CreatedById` (string, ID del usuario)
- `UpdatedAt` (DateTime?, UTC, nullable)
- `UpdatedById` (string?, nullable)

## DbContext con auditoria automatica

El DbContext debe:

1. Recibir `ICurrentUserService` por inyeccion de dependencias.
2. Sobrescribir `SaveChangesAsync` para recorrer `ChangeTracker.Entries<IAuditable>()`.
3. En estado `Added`: asignar `CreatedAt` y `CreatedById`.
4. En estado `Modified`: asignar `UpdatedAt` y `UpdatedById`.
5. Usar `DateTime.UtcNow` siempre (nunca `DateTime.Now`).
6. Llamar `ApplyConfigurationsFromAssembly` en `OnModelCreating` para cargar todas las configuraciones `IEntityTypeConfiguration<T>` del ensamblado.

## Soft Delete

Aplicar global query filter `HasQueryFilter(e => !e.IsDeleted)` en las entidades que implementen soft delete. Recordar que los filtros globales se pueden ignorar con `.IgnoreQueryFilters()` cuando sea necesario.

## Configuraciones Fluent API

Crear una clase de configuracion separada por entidad (`IEntityTypeConfiguration<T>`), ubicada en `Infrastructure/Persistence/Configurations/`.

Convenciones obligatorias:

- Especificar nombre de tabla con `ToTable("NombrePlural")`.
- Usar `UseIdentityColumn()` para primary keys auto-incrementales.
- Definir `HasMaxLength` en todas las propiedades string.
- Convertir enums a string con `HasConversion<string>()` y especificar `HasMaxLength`.
- Nombrar indices explicitamente: `HasDatabaseName("IX_Tabla_Columna")`.
- Usar `IncludeProperties` en indices compuestos cuando beneficie las queries frecuentes.
- Especificar `OnDelete` explicitamente en todas las relaciones.

## Queries

- Usar `AsNoTracking()` en toda query de solo lectura.
- Preferir proyeccion con `Select` a DTOs en lugar de `Include` cuando no se necesita la entidad completa.
- Usar `ExecuteUpdateAsync` / `ExecuteDeleteAsync` para operaciones masivas (EF Core 7+).
- Usar split queries (`AsSplitQuery()`) cuando haya multiples `Include` para evitar explosion cartesiana.
