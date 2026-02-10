---
name: iso27001-security
description: Aplica controles de seguridad ISO 27001:2022 segun los requisitos especificos de RSM Peru. Se activa cuando se implementa autenticacion, autorizacion, audit logging, proteccion de datos sensibles, validacion de entrada o revision de seguridad en endpoints de aplicaciones empresariales .NET/Azure.
---

# Controles ISO 27001:2022 para RSM Peru

## Control de acceso (A.5.15 a A.5.18)

Usar autenticacion con Azure AD como proveedor de identidad. No implementar autenticacion propia.

Configurar autorizacion basada en roles con las siguientes politicas especificas de RSM Peru:

- Politica `RequireAuditorClaim`: requiere claim `role` con valor `Auditor`. Solo auditores acceden a logs.
- Roles del sistema: `Auditor`, `Admin`, `Consultant`, `Viewer`.
- Issuer de tokens JWT: `rsmperu-api`.
- Audience de tokens JWT: `rsmperu-app`.
- Tiempo de vida maximo de token: 8 horas.
- ClockSkew: `TimeSpan.Zero` (sin tolerancia).

## Audit logging (A.8.15)

Registrar todas las operaciones mediante un middleware de auditoria. Usar el siguiente esquema obligatorio para la tabla `AuditLogs`:

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| Id | INT IDENTITY PK | Identificador |
| UserId | NVARCHAR(450) NOT NULL | Quien ejecuto la accion |
| Action | NVARCHAR(500) NOT NULL | Metodo HTTP + ruta |
| EntityType | NVARCHAR(256) | Tabla afectada |
| EntityId | NVARCHAR(256) | Registro afectado |
| Timestamp | DATETIME2 NOT NULL | Momento UTC de la accion |
| IpAddress | NVARCHAR(45) | IP de origen |
| Changes | NVARCHAR(MAX) | Detalle de cambios en JSON |
| RetentionDate | DATETIME2 NOT NULL | DEFAULT DATEADD(YEAR, 7, GETUTCDATE()) |

### Retencion de 7 anos

La politica de retencion de RSM Peru exige conservar audit logs por 7 anos. Implementar:

1. Columna `RetentionDate` con valor por defecto de 7 anos desde la fecha de creacion.
2. Procedimiento almacenado `sp_ArchiveOldAuditLogs` que mueva registros vencidos a tabla `AuditLogsArchive` y luego los elimine de `AuditLogs`.

## Proteccion de datos (A.8.11)

Usar ASP.NET Core Data Protection API para cifrar columnas sensibles.

**Purpose string obligatorio:** `RSMPeru.DataProtection`

Ejemplo de uso:
```
provider.CreateProtector("RSMPeru.DataProtection")
```

Almacenar columnas cifradas como `varbinary(max)` en SQL Server.

## Sesiones y tokens (A.8.3)

Configurar JWT Bearer con estos parametros fijos de RSM Peru:

- ValidateIssuerSigningKey: true
- ValidIssuer: `rsmperu-api`
- ValidAudience: `rsmperu-app`
- ValidateLifetime: true
- ClockSkew: TimeSpan.Zero
- Expiracion: 8 horas

## Validacion de entrada (A.8.12)

Usar Data Annotations en todos los comandos/DTOs. Nunca usar raw SQL con interpolacion de strings. Usar Entity Framework para prevenir SQL injection.

## Hashing de passwords (A.8.5)

Si no se usa Azure AD para un flujo especifico, usar BCrypt o PBKDF2. Nunca almacenar passwords en texto plano.

## HTTPS (A.8.9)

Forzar HTTPS en todos los ambientes que no sean desarrollo local. En Azure App Service, activar SSL Settings > HTTPS Only.

## Checklist de seguridad RSM Peru

Verificar que todo proyecto cumpla con:

- [ ] Autenticacion via Azure AD
- [ ] Roles: Auditor, Admin, Consultant, Viewer
- [ ] Audit logging con esquema completo (ver tabla arriba)
- [ ] Retencion de logs: 7 anos
- [ ] Secrets almacenados en Azure Key Vault (nunca en appsettings.json)
- [ ] HTTPS obligatorio
- [ ] Data Protection con purpose string `RSMPeru.DataProtection`
- [ ] Tokens JWT: issuer `rsmperu-api`, audience `rsmperu-app`, 8h max
- [ ] Validacion de entrada con Data Annotations
- [ ] Passwords hasheados con BCrypt o PBKDF2
- [ ] SQL injection prevenido (EF, nunca raw SQL interpolado)
- [ ] Application Insights habilitado para monitoreo
