---
name: aspnet-enterprise
description: "Desarrolla Web APIs en ASP.NET Core con Clean Architecture para proyectos de RSM Peru. Usar cuando (1) se crea o modifica una API en ASP.NET Core, (2) se implementa Clean Architecture con MediatR, (3) se configura Entity Framework Core, (4) se integra con Azure (App Service, Key Vault, SQL Database, Application Insights), (5) se aplican controles de seguridad y compliance ISO 27001, (6) se trabaja con auditorias o sistemas empresariales de RSM Peru."
---

# Desarrollo ASP.NET Core Enterprise - RSM Peru

## Estructura de proyecto (Clean Architecture)

Respetar esta estructura de capas en todos los proyectos:

```
Solution/
  Domain/
    Entities/
    ValueObjects/
    Interfaces/
  Application/
    UseCases/
    DTOs/
    Validators/
  Infrastructure/
    Data/
    Repositories/
    Services/
  API/
    Controllers/
    Middleware/
```

- `Domain`: entidades, value objects, interfaces de repositorio. Sin dependencias externas.
- `Application`: casos de uso con MediatR (commands/queries), DTOs, validadores FluentValidation.
- `Infrastructure`: implementaciones de EF Core, servicios Azure, repositorios concretos.
- `API`: controllers delgados, middleware, configuracion de startup.

## Convenciones RSM Peru

### Nombres

- Metodos async: sufijo `Async` (ejemplo: `GetWorkItemAsync`).
- Campos privados: prefijo `_` (ejemplo: `_context`, `_logger`).
- Interfaces: prefijo `I` (ejemplo: `IWorkItemRepository`).
- DTOs: sufijo `Dto` (ejemplo: `WorkItemDto`).
- Commands/Queries MediatR: sufijo `Command` o `Query` (ejemplo: `CreateWorkItemCommand`).
- Rutas API: `api/v1/[controller]`, siempre versionadas.

### Controllers

- Siempre delgados: solo reciben request, envian a MediatR, devuelven respuesta.
- Siempre con `[ApiController]`, `[Authorize]` y `[Route("api/v1/[controller]")]`.
- Documentar con `[ProducesResponseType]` en cada endpoint.

### Wrapper de respuesta estandar

Todos los endpoints devuelven este wrapper:

```csharp
public class ApiResponse<T>
{
    public bool Success { get; set; }
    public T Data { get; set; }
    public string Message { get; set; }
    public List<string> Errors { get; set; }
}
```

### Entity Framework Core

- Usar siempre configuracion explicita con `IEntityTypeConfiguration<T>` (nunca data annotations).
- Usar `.AsNoTracking()` en todas las consultas de solo lectura.
- Incluir `CancellationToken` en todos los metodos async.
- Timestamps: usar `GETUTCDATE()` como default en SQL Server.
- Relaciones: especificar `OnDelete` explicitamente (preferir `Restrict` sobre `Cascade`).
- No usar `ToList()` cuando se necesita `FirstOrDefault()`.

### Validacion

- Usar FluentValidation para todos los commands y queries.
- Registrar validadores automaticamente con `AddValidatorsFromAssembly`.

## Integracion Azure

### Key Vault

- En produccion, cargar secretos desde Azure Key Vault con `DefaultAzureCredential`.
- Nunca colocar secretos en `appsettings.json`.

### SQL Database

- Usar Managed Identity para conexion. Formato de connection string:

```
Server=tcp:{servidor}.database.windows.net;Database={base};Authentication=Active Directory Default;
```

- No usar connection strings con usuario y password.

### Application Insights

- Habilitar con `AddApplicationInsightsTelemetry()`.
- Usar structured logging con `ILogger<T>`. Ejemplo: `_logger.LogInformation("Work item {Id} retrieved", id)`.
- No concatenar strings en mensajes de log.

## Seguridad (ISO 27001)

### Autenticacion

- Usar Azure AD con `AddMicrosoftIdentityWebApi`.
- Definir politicas de autorizacion por roles del negocio (ejemplo: `RequireAuditorRole` para roles `Auditor` y `Admin`).

### Audit logging

- Implementar middleware de auditoria que registre: usuario, accion (metodo HTTP + ruta) y timestamp UTC.
- Todo request autenticado debe quedar registrado en logs.

### Reglas generales

- No exponer stack traces en produccion.
- Validar todos los inputs con FluentValidation antes de procesarlos.
- Usar HTTPS exclusivamente.

## Reglas estrictas

- No usar `var` para tipos primitivos (`string`, `int`, `bool`).
- No hacer queries sin `.AsNoTracking()` en lecturas.
- No omitir `CancellationToken` en metodos async.
- No usar metodos sincronos para acceso a datos.
- No colocar logica de negocio en controllers.
