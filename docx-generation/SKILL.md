---
name: docx-generation
description: "Genera documentos Word (.docx) y correos HTML para RSM Peru usando pandoc con plantilla corporativa. Usar cuando se cree un documento Word, informe, propuesta, reporte o entregable para un cliente de RSM Peru. Tambien cuando se genere un correo electronico en formato HTML o una presentacion PowerPoint (.pptx)."
---

# Generacion de documentos RSM Peru

## Antes de generar

Preguntar a Mau el nombre del **cliente** si no se ha mencionado en la conversacion. El cliente se usa en `subtitle`.

## Herramientas

- Documentos Word (.docx): usar **pandoc** (ya instalado). No usar python-docx ni crear entornos virtuales.
- Presentaciones (.pptx): usar **python-pptx** (instalado en `/opt/homebrew/bin/python3.14`).
- Diagramas: usar **mermaid-cli** (`mmdc`) o **Eraser MCP**.

## Plantilla

Plantilla V2 basada en la plantilla oficial RSM 2024:

```
/Users/msouga/Library/Group Containers/UBF8T346G9.Office/User Content.localized/Templates.localized/RSM_Peru_Plantilla_v2.dotx
```

Repositorio de plantillas: `~/Trabajo/RSM/RSM Peru/Plantillas y Documentacion/`

Caracteristicas de la plantilla V2 (aplicadas por fix-docx-v2.py):

- **Tema**: RSM 2024 con colores corporativos (42+ tonalidades)
- **Fuentes**: Arial (tema major y minor)
- **Pagina**: A4 con margenes oficiales RSM
- **Portada**: Fondo azul oscuro (#00153D), onda PoP, logo RSM grande, campos DRAFT + endorsement + titulo + subtitulo + fecha
- **Header**: Logo RSM en esquina superior derecha (paginas de contenido)
- **Footer**: Titulo del documento | RSM Peru S.A.C. | Pagina N / Total (paginas de contenido)
- **Heading 1**: Arial 26pt, azul RSM (#009CDE), salto de pagina antes
- **Heading 2**: Arial Bold, azul RSM
- **Body Text**: Arial 9pt, gris (#63666A)
- **Tablas**: Bordes azules superior/inferior, fila header azul con texto blanco
- **Listas numeradas**: Nivel 1 = numeros, nivel 2 = letras, nivel 3 = romanos (requiere fix-docx.py)
- **Source Code**: Consolas 8pt, fondo gris claro
- **Propiedades**: Autor del sistema, compania RSM Peru, categoria inferida, keywords auto-generadas

## Ubicacion de archivos

El .md fuente y el .docx generado van en la **misma subcarpeta** dentro de `documentacion/`, segun el proposito del documento:

| Proposito | Carpeta |
|-----------|---------|
| Informes, reportes, diagnosticos | `documentacion/informes/` |
| Documentos de arquitectura | `documentacion/arquitectura/` |
| Manuales de uso, guias | `documentacion/manuales/` |
| Propuestas tecnicas y comerciales | `documentacion/propuestas/` |
| Documentos ISO 27001, politicas | `documentacion/seguridad/` |
| Correos electronicos | `documentacion/emails/` |

Ejemplo: `documentacion/informes/ejecucion-noche1.md` genera `documentacion/informes/ejecucion-noche1.docx`.

Los diagramas van en `documentacion/diagramas/` y se referencian con ruta relativa `../diagramas/` desde cualquier subcarpeta de `documentacion/`.

## Frontmatter YAML

```yaml
---
title: "Titulo del Documento"
subtitle: "Descripcion o subtitulo - Cliente Y"
date: "Mes Ano"
status: "DRAFT"
endorsement: "INTERNAL USE ONLY"
category: "Propuesta"
keywords: "palabra1; palabra2; palabra3"
---
```

| Campo | Requerido | Donde aparece |
|-------|-----------|---------------|
| `title` | Si | Portada (blanco 32pt) + campo TITLE en footer |
| `subtitle` | Si | Portada (blanco 18pt) + propiedad Asunto |
| `date` | Si | Portada (blanco 12pt) |
| `status` | No | Portada (blanco 24pt, default: "DRAFT"). Campo DOCPROPERTY editable en Word |
| `endorsement` | No | Portada (azul RSM 18pt, default: "INTERNAL USE ONLY"). Campo DOCPROPERTY editable |
| `category` | No | Propiedad Categoria. Si no se especifica, se infiere del titulo |
| `keywords` | No | Propiedad Palabras clave. Si no se especifica, se auto-generan (separadas por `;`) |

No incluir `author` en el YAML. fix-docx-v2.py lo completa automaticamente con el nombre del usuario del sistema operativo.

## Tabla de Contenido (TOC)

Si el documento necesita TOC, insertar despues del frontmatter. Poner salto de pagina ANTES (para separar de la caratula) pero NO despues (el primer `#` ya genera nueva pagina).

````markdown
```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
<w:p>
  <w:pPr><w:pStyle w:val="TtuloTDC" /></w:pPr>
  <w:r><w:t>Tabla de contenido</w:t></w:r>
</w:p>
<w:p>
  <w:r><w:fldChar w:fldCharType="begin" w:dirty="true" /></w:r>
  <w:r><w:instrText xml:space="preserve"> TOC \o "1-3" \h \z \u </w:instrText></w:r>
  <w:r><w:fldChar w:fldCharType="separate" /></w:r>
  <w:r><w:t>Actualizar este campo para ver la tabla de contenido</w:t></w:r>
  <w:r><w:fldChar w:fldCharType="end" /></w:r>
</w:p>
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# 1. Primer capitulo
````

Al abrir en Word: clic derecho en la TOC > "Actualizar campo" > "Actualizar toda la tabla". O Ctrl+A y luego F9.

## Estructura del contenido

- No usar `#` para el titulo (ya sale del frontmatter YAML). Capitulos empiezan en `#`.
- Numerar encabezados: `# 1. Capitulo`, `## 1.1. Subcapitulo`, `### 1.1.1. Sub-sub`.
- Linea en blanco obligatoria antes de listas (`-`) y entre parrafos. Sin ella, pandoc NO renderiza la lista y la colapsa en el parrafo anterior. Especialmente critico despues de un parrafo que termina en `:` seguido de items con `-`.
- No mezclar `*` literal dentro de `**` negrita (pandoc confunde asteriscos). Usar "wildcard" en vez de `*`.
- No usar emojis, en-dash, em-dash ni lineas horizontales (`---`).
- Todo el texto en espanol con tildes correctas (a, e, i, o, u), ene (n, N) y signos de apertura (!, ?). Verificar antes de generar el Word.

## Tablas para Word

- Columnas con valores numericos: alinear a la derecha con `---:` en el separador.
- Columnas de texto: alinear a la izquierda con `:---`.
- La alineacion aplica a toda la columna (encabezado + datos). No se puede diferenciar encabezado vs datos en Markdown.
- Ejemplo:

```markdown
| Elemento | Cantidad | Descripcion |
|:---------|----------:|:------------|
| Procesos | 110 | Procesos activos |
```

- Maximo 6-7 columnas por tabla para que quepan en A4 vertical.
- No usar negrita (`**`) dentro de celdas de tabla (puede romper el formato en Word).

## Diagramas e imagenes

### Centrado de imagenes y captions

Pandoc no centra imagenes automaticamente en Word. Usar bloques raw openxml para centrar la imagen y agregar un caption en italica debajo. El alt text de la imagen debe estar vacio `![]()` para que pandoc no genere un caption duplicado alineado a la izquierda.

````markdown
```{=openxml}
<w:p><w:pPr><w:jc w:val="center"/></w:pPr></w:p>
```

![](../diagramas/mi-diagrama.png)

```{=openxml}
<w:p><w:pPr><w:jc w:val="center"/></w:pPr><w:r><w:rPr><w:i/><w:sz w:val="20"/></w:rPr><w:t>Figura N: Descripcion del diagrama</w:t></w:r></w:p>
```
````

### Herramientas para diagramas

1. **Mermaid** (Gantt, flowcharts simples, secuencia):
   - Crear archivo `.mmd` en `documentacion/diagramas/`
   - Generar PNG: `mmdc -i diagrama.mmd -o diagrama.png -b white -w 1200`

2. **Eraser MCP** (arquitectura cloud, ERDs, flowcharts complejos, BPMN):
   - Usar las herramientas renderFlowchart, renderCloudArchitectureDiagram, renderSequenceDiagram, etc.
   - Copiar el PNG generado (de `.eraser/scratchpad/`) a `documentacion/diagramas/`

### Ubicacion

Guardar todos los diagramas en `documentacion/diagramas/` y referenciar con ruta relativa `../diagramas/` desde cualquier subcarpeta de `documentacion/`.

## Comando pandoc

Desde la raiz del proyecto. Siempre incluir `--resource-path` con la carpeta del documento y la de diagramas.

Paso 1: generar el docx con pandoc (usar plantilla Con_Footer, NO la v2 directamente):

```bash
pandoc documentacion/informes/mi-informe.md \
  --reference-doc="/Users/msouga/Library/Group Containers/UBF8T346G9.Office/User Content.localized/Templates.localized/RSM_Peru_Plantilla_Con_Footer.dotx" \
  --resource-path=documentacion/informes:documentacion/diagramas \
  -o documentacion/informes/mi-informe.docx
```

Paso 2: post-procesar con fix-docx-v2.py (portada RSM + header/footer + propiedades):

```bash
python3 "$HOME/Trabajo/RSM/RSM Peru/Plantillas y Documentacion/plantillas/fix-docx-v2.py" documentacion/informes/mi-informe.docx
```

**Importante**: El paso 2 es SIEMPRE necesario. Inyecta la portada corporativa, header con logo, footer con titulo/paginacion, y propiedades del documento. La plantilla v2 (.dotx) NO es compatible con pandoc por el SVG en headers.

Paso 3: post-procesar listas numeradas (corrige formato a 1. a. i.):

```bash
python3 "$HOME/Trabajo/RSM/RSM Peru/Plantillas y Documentacion/plantillas/fix-docx.py" documentacion/informes/mi-informe.docx
```

**Importante**: El paso 3 es necesario solo si el documento tiene listas numeradas con mas de un nivel. Pandoc genera todos los niveles con numeros decimales; fix-docx.py los corrige a numeros, letras y romanos.

## Footer automatico

La plantilla V2 incluye un footer en todas las paginas excepto la primera (caratula):

- Izquierda: titulo del documento (campo TITLE del YAML)
- Izquierda (continuacion): " | RSM Peru S.A.C."
- Derecha: numero de pagina / total de paginas
- Arriba: linea azul separadora

El titulo del footer se actualiza automaticamente desde el campo `title` del YAML.

## Correos electronicos

Generar correos en formato **HTML** (no Word). Guardar en `documentacion/emails/`.

Estructura del HTML:

```html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<style>
body { font-family: Arial, sans-serif; font-size: 11pt; color: #333333; line-height: 1.6; max-width: 680px; margin: 0 auto; padding: 20px; }
b, strong { color: #000000; }
ul { margin-top: 4px; }
li { margin-bottom: 4px; }
.asunto { font-size: 12pt; color: #666666; margin-bottom: 16px; border-bottom: 1px solid #cccccc; padding-bottom: 8px; }
</style>
</head>
<body>
<div class="asunto"><b>Asunto:</b> Texto del asunto</div>
<p>Hola [nombres],</p>
<p>Contenido del correo...</p>
<p>Saludos,<br>Mau</p>
</body>
</html>
```

Reglas:

- Fuente: Arial, 11pt.
- Color de texto: `#333333`, negritas en `#000000`.
- Incluir el **Asunto** visible al inicio del body.
- Texto en espanol con tildes correctas.
- Abrir en navegador para copiar (Cmd+A, Cmd+C) y pegar en Outlook (Cmd+V). Esto preserva negritas, listas y formato uniforme.

## Revision ortografica

Antes de generar el .docx, ejecutar revision ortografica sobre el .md fuente.

### Paso 1: hunspell (deteccion rapida)

```bash
sed '1{/^---$/,/^---$/d}' ARCHIVO.md | \
  sed 's|https\?://[^ ]*||g' | \
  sed 's|`[^`]*`||g' | \
  sed '/^```/,/^```/d' | \
  hunspell -d es_ES -l | \
  sort -u
```

Ignorar falsos positivos: nombres de productos (Azure, Microsoft, Entra, etc.), acronimos (WAF, MFA, DLP, etc.), terminos tecnicos en ingles de uso comun en IT, codigos de examen, URLs y rutas.

### Paso 2: revision inteligente

Leer el .md completo y revisar:

- **Tildes**: faltantes o incorrectas (a, e, i, o, u)
- **Ene**: uso correcto de n
- **Puntuacion**: signos de apertura (!, ?), comas, puntos
- **Gramatica**: concordancia genero/numero, preposiciones, conjugaciones
- **Estilo** (solo reportar): oraciones largas, repeticiones, gerundios mal usados

### Paso 3: correccion

Presentar resultados en tabla:

```
| Linea | Error | Correccion | Tipo |
|-------|-------|------------|------|
| 11    | certificacion | certificacion | Tilde |
```

Preguntar a Mau: (1) aplicar todas, (2) revisar una por una, o (3) solo el reporte.

## Checklist antes de entregar

1. Ejecutar revision ortografica (hunspell + revision inteligente) sobre el .md
2. Verificar que todas las tablas con columnas numericas tienen `---:` en el separador
3. Verificar linea en blanco antes de cada lista con `-`
4. Verificar que las imagenes tienen alt text vacio `![]()` si usan caption raw openxml
5. Verificar que el frontmatter tiene al menos: title, subtitle, date
6. Ejecutar fix-docx-v2.py (portada + header/footer + propiedades)
7. Ejecutar fix-docx.py si hay listas numeradas con subniveles
8. Abrir el .docx en Word y actualizar campos: Ctrl+A, F9
