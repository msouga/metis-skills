---
name: project-docs
description: Inicializa la estructura de documentacion estandar para proyectos de RSM Peru. Usar cuando se diga "inicializa docs", "inicializa documentacion", "project-docs" o se necesite organizar la carpeta de documentacion de un proyecto nuevo o existente.
---

# Skill: project-docs

Inicializa y organiza la estructura de documentacion estandar para proyectos RSM Peru.

## Estructura de directorios

Al ejecutar este skill, crear la siguiente estructura dentro del directorio del proyecto:

```
proyecto/
  CLAUDE.md
  .claude/
  documentacion/
    informes/          <- reportes, diagnosticos, informes de ejecucion (.md + .docx)
    arquitectura/      <- diseno tecnico, documentos de arquitectura (.md + .docx)
    manuales/          <- manuales de uso, guias de operacion (.md + .docx)
    propuestas/        <- propuestas tecnicas y comerciales (.md + .docx)
    seguridad/         <- documentos ISO 27001, politicas, evaluaciones (.md + .docx)
    diagramas/         <- .png, .drawio (generados con Eraser u otros)
    referencias/       <- .pdf, .xlsx externos, contratos, manuales de terceros
    emails/            <- .html de correos generados
    capturas/          <- screenshots de pantallas
  src/                 (codigo, cuando nace la fase de desarrollo)
    backend/
    frontend/
    infra/
  scripts/             (scripts de utilidad del proyecto)
```

### Reglas de subcarpetas

- Solo crear las subcarpetas que el proyecto necesite (no crear vacias).
- Si un proyecto solo tiene informes, crear unicamente `documentacion/informes/`.
- Las subcarpetas se agregan conforme se generan documentos.

## Convenciones de nombres

| Tipo | Ubicacion | Ejemplo |
|------|-----------|---------|
| Informe/reporte MD | `documentacion/informes/` | ejecucion-noche1.md |
| Informe/reporte DOCX | `documentacion/informes/` | ejecucion-noche1.docx |
| Documento de arquitectura | `documentacion/arquitectura/` | arquitectura-gateway.md |
| Manual de uso | `documentacion/manuales/` | manual-operacion.md |
| Propuesta tecnica | `documentacion/propuestas/` | propuesta-migracion.md |
| Presentacion PPTX | `documentacion/propuestas/` | propuesta-migracion.pptx |
| Documento de seguridad | `documentacion/seguridad/` | politica-acceso.md |
| Diagrama PNG | `documentacion/diagramas/` | arquitectura-cloud.png |
| Email HTML | `documentacion/emails/` | seguimiento-kickoff.html |
| PDF de referencia | `documentacion/referencias/` | manual-tecnico-bcp.pdf |
| Excel de referencia | `documentacion/referencias/` | estructuras-h2h-2025.xlsx |
| Capturas de pantalla | `documentacion/capturas/` | configuracion-vpn.png |

### Reglas de nombres de archivo

- Usar **kebab-case** (minusculas, separado por guiones).
- Nombres descriptivos, sin prefijos de tipo (la carpeta ya lo indica).
- MD y DOCX del mismo documento van en la **misma carpeta** con el **mismo nombre**: `ejecucion-noche1.md` + `ejecucion-noche1.docx`.
- PPTX va junto al MD en la carpeta de proposito correspondiente.

## Comportamiento al ejecutar

### Proyecto nuevo (no existe documentacion/)

1. Preguntar a Mau que tipos de documentos espera generar en el proyecto.
2. Crear `documentacion/` con solo las subcarpetas necesarias.
3. Imprimir resumen de lo creado.

### Proyecto existente (archivos sueltos o en docs/)

1. Crear la estructura `documentacion/` si no existe.
2. Analizar los archivos existentes e inferir su proposito por nombre y contenido.
3. Preguntar a Mau cuando el proposito no sea claro (ej: "Este archivo parece un informe, ¿va en informes/?").
4. Mover archivos a sus carpetas correspondientes:
   - `*.md` (excepto CLAUDE.md y README.md) -> subcarpeta de proposito en `documentacion/`
   - `*.docx` -> misma subcarpeta que su .md correspondiente, o inferir proposito
   - `*.pptx` -> subcarpeta de proposito (generalmente `propuestas/`)
   - `*.pdf` -> `documentacion/referencias/`
   - `*.xlsx`, `*.xls` -> `documentacion/referencias/`
   - `*.html` (correos) -> `documentacion/emails/`
   - `*.png`, `*.jpg` de diagramas -> `documentacion/diagramas/`
   - `*.png`, `*.jpg` de capturas -> `documentacion/capturas/`
5. Actualizar rutas relativas dentro de los archivos .md movidos.
6. Imprimir resumen de lo movido.

### Migracion desde docs/ existente

Si el proyecto ya tiene estructura `docs/` con `sources/`, `deliverables/`, etc.:

1. Crear `documentacion/` con subcarpetas de proposito.
2. Mover contenido de `docs/sources/*.md` a la subcarpeta de proposito correspondiente.
3. Mover contenido de `docs/deliverables/*.docx` junto a su .md en la misma subcarpeta.
4. Mover `docs/diagrams/` a `documentacion/diagramas/`.
5. Mover `docs/references/` a `documentacion/referencias/`.
6. Mover `docs/emails/` a `documentacion/emails/`.
7. Eliminar `docs/` una vez vacia.
8. Actualizar rutas relativas en archivos .md.

### Reglas importantes

- NUNCA mover `CLAUDE.md`, `README.md`, `.claude/`, `.gitignore` u otros archivos de configuracion del proyecto.
- No crear `src/` ni `scripts/` automaticamente; solo crear `documentacion/` y sus subcarpetas.
- Si un archivo ya esta en la ubicacion correcta dentro de `documentacion/`, no moverlo.

## Integracion con otros skills

- **docx-generation**: El .md fuente y el .docx generado van en la misma subcarpeta de proposito. Los diagramas se referencian con ruta relativa `../diagramas/` desde cualquier subcarpeta de `documentacion/`.
- **eraser-diagrams**: Los PNGs generados deben guardarse en `documentacion/diagramas/`.
- **aspnet-enterprise**: El codigo va en `src/backend/`, separado de la documentacion.
- **angular-enterprise**: El codigo frontend va en `src/frontend/`.

## Comando pandoc de referencia

Desde la raiz del proyecto (ejemplo para un informe):

```bash
pandoc documentacion/informes/mi-informe.md \
  --from markdown \
  --to docx \
  --reference-doc="/Users/msouga/Library/Group Containers/UBF8T346G9.Office/User Content.localized/Templates.localized/RSM_Peru_Plantilla_Con_Footer.dotx" \
  --resource-path=documentacion/informes:documentacion/diagramas \
  --toc \
  --toc-depth=3 \
  -o documentacion/informes/mi-informe.docx
```
