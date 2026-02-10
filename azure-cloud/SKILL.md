---
name: azure-cloud
description: Convenciones de Azure para proyectos de RSM Peru, incluyendo nombres de recursos, Key Vault, Managed Identity, deployment slots y pipelines de Azure DevOps. Usar cuando se configure infraestructura Azure, pipelines CI/CD, secrets o monitoreo para clientes de RSM Peru.
---

# Convenciones Azure de RSM Peru

## Nomenclatura de recursos

Usar estos patrones de nombres para todos los recursos Azure de RSM Peru:

| Recurso | Patron | Ejemplo |
|---------|--------|---------|
| Resource Group | `rsmperu-rg` o `rsmperu-{proyecto}-rg` | `rsmperu-rg` |
| Key Vault | `rsmperu-kv` o `rsmperu-{proyecto}-kv` | `rsmperu-kv` |
| App Service | `rsmperu-{app}` | `rsmperu-api` |
| SQL Server | `rsmperu.database.windows.net` | |
| Storage Account | `rsmperustorage` (sin guiones) | |

## Key Vault

- URL base: `https://rsmperu-kv.vault.azure.net/`
- Configurar en `appsettings.json` bajo la clave `KeyVault:Url`.
- En produccion, cargar secretos con `AddAzureKeyVault` usando `DefaultAzureCredential`.
- En desarrollo, no cargar Key Vault (condicionar con `IsDevelopment()`).

## Managed Identity

Regla principal: no usar connection strings con credenciales en produccion.

- Para SQL: usar `Authentication=Active Directory Default` en la cadena de conexion.
- Para Key Vault, Storage y otros servicios: usar `DefaultAzureCredential()`.
- Otorgar permisos al App Service con `az keyvault set-policy` usando el object-id de la Managed Identity.

## Deployment Slots

Flujo obligatorio para produccion:

1. Crear slot `staging` en el App Service.
2. Deployar al slot `staging`.
3. Validar en staging.
4. Ejecutar swap de `staging` a `production`.

Nombres de App Service para los comandos: `rsmperu-api`, resource group `rsmperu-rg`.

## Azure DevOps Pipeline

Usar la plantilla base en `references/azure-pipelines-template.yml` de este skill.

Estructura requerida:
- Stage `Build`: restore, build, test con cobertura, publish de artefactos.
- Stage `Deploy`: deployment job al slot `staging` del App Service.
- Pool: `windows-latest`.
- .NET version: `8.x` (ajustar segun proyecto).

## Application Insights

- Siempre agregar `AddApplicationInsightsTelemetry()` al builder de la aplicacion.
- Usar `TelemetryClient` para metricas y eventos custom.
- El tracking de dependencias HTTP y SQL es automatico.

## Reglas estrictas

- No poner secrets en appsettings.json ni en codigo fuente.
- No usar connection strings con password en produccion (usar Managed Identity).
- No deployar directo a produccion (usar staging slots + swap).
- No omitir Application Insights en ningun proyecto.
- No hardcodear URLs de Azure (usar configuracion).
