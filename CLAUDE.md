# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proposito

Este directorio contiene los skills de Claude Code para proyectos de RSM Peru. Los skills siguen las [best practices oficiales de Anthropic](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) y fueron validados con `quick_validate.py` del skill-creator.

## Contenido

7 subdirectorios de skills, cada uno con su `SKILL.md` y opcionalmente un directorio `references/` con material de consulta.

## Skills

Cada skill esta instalado en `~/.claude/skills/<nombre>/SKILL.md`. Solo contienen convenciones y decisiones especificas de RSM Peru; no repiten conocimiento que Claude ya posee.

| Skill | Proposito | Lineas |
|-------|-----------|--------|
| aspnet-enterprise | Clean Architecture con MediatR, wrapper ApiResponse, convenciones de nombres, integracion Azure/ISO 27001 | 128 |
| angular-enterprise | Estructura de proyecto, features de dominio (work-items, audits, reports), patron de servicios con ApiResponse | 103 |
| ef-core-patterns | IAuditable, auditoria en SaveChangesAsync, Fluent API, migraciones multi-proyecto | 80 |
| sql-optimization | Nomenclatura de tablas/indices/SPs, patron de paginacion, transacciones con auditoria | 75 |
| azure-cloud | Nomenclatura de recursos, Key Vault, Managed Identity, deployment slots, pipeline template | 68 |
| azure-architecture | Patron de referencia, tiers dev/prod, costos, checklist pre-produccion | 74 |
| iso27001-security | Controles A.5/A.8, esquema AuditLogs, retencion 7 anos, roles, JWT config | 95 |
| docx-generation | Documentos Word con pandoc, plantilla RSM, tablas, correos HTML, presentaciones pptx | 91 |

## Formato de los skills (reglas oficiales)

- Frontmatter YAML: solo `name` y `description`. No usar `allowed-tools`, `model` ni otros campos.
- `name`: minusculas, numeros y guiones. Maximo 64 caracteres.
- `description`: tercera persona, maximo 1024 caracteres. Incluir que hace el skill y cuando usarlo.
- Cuerpo: maximo 500 lineas. Mover contenido extenso a `references/`.
- No incluir README, CHANGELOG ni documentacion auxiliar dentro de un skill.

## Validacion

Para validar un skill con el script oficial:

```bash
python3 ~/.claude/skills/skill-creator/scripts/quick_validate.py <directorio-del-skill>
```

## MCP Servers recomendados

| Servidor | Proposito |
|----------|-----------|
| azure-devops | Work items, sprints, pipelines, pull requests |
| context7 | Documentacion actualizada de Angular, .NET, TypeScript, RxJS, Azure SDKs |
| microsoft-learn | Documentacion oficial de Microsoft, guias de Azure, tutoriales |
| github | Gestion de codigo, PRs, issues |

## Stack de RSM Peru

- Frontend: Angular + TypeScript
- Backend: ASP.NET Core Web API + C#
- ORM: Entity Framework Core 8.0
- Base de datos: SQL Server (Azure SQL Database)
- Cloud: Azure (App Service, Key Vault, Application Insights)
- CI/CD: Azure DevOps Pipelines
- Auth: Azure AD + JWT
- Compliance: ISO 27001
