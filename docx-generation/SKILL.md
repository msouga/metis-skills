---
name: docx-generation
description: "Genera documentos Word (.docx) y correos HTML para RSM Peru usando pandoc con plantilla corporativa. Usar cuando se cree un documento Word, informe, propuesta, reporte o entregable para un cliente de RSM Peru. Tambien cuando se genere un correo electronico en formato HTML o una presentacion PowerPoint (.pptx)."
---

# Generacion de documentos RSM Peru

## Antes de generar

Preguntar a Mau el nombre del **cliente** si no se ha mencionado en la conversacion. El cliente se usa en `subtitle` y queda registrado en la metadata del documento.

## Herramientas

- Documentos Word (.docx): usar **pandoc** (ya instalado). No usar python-docx ni crear entornos virtuales.
- Presentaciones (.pptx): usar **python-pptx** (instalado en `/opt/homebrew/bin/python3.14`).
- Diagramas: usar **mermaid-cli** (`mmdc`).

## Plantilla

```
/Users/msouga/Trabajo/RSM/Plantillas y Documentacion/plantillas/RSM_Peru_Plantilla_Con_Footer.dotx
```

## Frontmatter YAML (obligatorio)

```yaml
---
title: "Titulo del Documento"
subtitle: "Proyecto X - Cliente Y"
shorttitle: "Titulo Corto para Footer"
author: "RSM Peru"
date: "Mes Ano"
---
```

| Campo | Donde aparece |
|-------|---------------|
| `title` | Estilo Titulo (inicio del documento) |
| `subtitle` | Estilo Subtitulo (incluir nombre del cliente) |
| `shorttitle` | Footer izquierda |
| `author` | Propiedad Autor del documento |
| `date` | Propiedad Fecha del documento |

## Estructura del contenido

- No usar `#` para el titulo (ya sale del frontmatter YAML). Capitulos empiezan en `#`.
- Numerar encabezados: `# 1. Capitulo`, `## 1.1. Subcapitulo`, `### 1.1.1. Sub-sub`.
- Linea en blanco obligatoria antes de listas (`-`) y entre parrafos (pandoc no renderiza sin ella).
- No mezclar `*` literal dentro de `**` negrita (pandoc confunde asteriscos). Usar "wildcard" en vez de `*`.
- No usar emojis, en-dash, em-dash ni lineas horizontales (`---`).

## Tablas para Word

- Maximo 6-7 columnas por tabla para que quepan en hoja carta vertical.
- Abreviar encabezados de meses: Ago, Sep, Oct, Nov, Dic, Ene (sin ano si hay leyenda).
- Abreviar nombres largos de recursos Azure y poner leyenda al pie con los nombres completos.
- Quitar prefijo "Standard_" de SKUs de VM (dejar solo D2s_v3, B4ms, etc.).
- Abreviar: Deallocated -> Dealloc., Windows -> Win.
- No usar negrita (`**`) dentro de celdas de tabla (puede romper el formato en Word).
- Agrupar servicios con costo < $1/mes en fila "Otros" con leyenda.

## Diagramas

1. Crear archivo `.mmd` (Mermaid).
2. Generar PNG: `mmdc -i diagrama.mmd -o diagrama.png -b white -w 800`
3. Insertar en el Markdown: `![Descripcion del diagrama](ruta/diagrama.png)`

## Comando pandoc

```bash
pandoc documento.md \
  --reference-doc="/Users/msouga/Trabajo/RSM/Plantillas y Documentacion/plantillas/RSM_Peru_Plantilla_Con_Footer.dotx" \
  -o documento.docx
```

## Footer automatico

La plantilla incluye un footer con:
- Izquierda: valor del campo `shorttitle` del YAML.
- Derecha: numero de pagina / total de paginas.
- Arriba: linea horizontal separadora.

Al abrir el documento en Word, responder "Si" cuando pregunte si actualizar referencias para que el campo shorttitle se actualice.

## Correos electronicos

Generar correos en formato **HTML** (no Word):

- Fuente: Arial, 11pt.
- Color de texto: `#333333`, negritas en `#000000`.
- Abrir en navegador para copiar (Cmd+A, Cmd+C) y pegar en Outlook (Cmd+V).
- Esto preserva negritas, listas y formato uniforme sin mezcla de fuentes.
