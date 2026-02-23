# Skill: spell-check

Revisa la ortografía y gramática de documentos Markdown en español.

Combina dos niveles de revisión:
1. **hunspell** para detección rápida de errores ortográficos
2. **Revisión inteligente** con Claude para gramática, tildes faltantes, concordancia y estilo

## Uso

El usuario indica un archivo .md (o directorio) y opcionalmente el idioma. Por defecto se asume español (es_ES).

## Paso 1: Revisión con hunspell

Ejecutar hunspell sobre el archivo para obtener la lista de palabras no reconocidas:

```bash
# Extraer texto plano del markdown (sin YAML frontmatter, sin URLs, sin código)
sed '1{/^---$/,/^---$/d}' ARCHIVO.md | \
  sed 's|https\?://[^ ]*||g' | \
  sed 's|`[^`]*`||g' | \
  sed '/^```/,/^```/d' | \
  hunspell -d es_ES -l | \
  sort -u
```

### Filtrar falsos positivos

Ignorar automáticamente:
- Nombres propios de productos y servicios (Azure, Microsoft, Entra, Sentinel, Purview, Defender, etc.)
- Códigos de examen (AZ-104, SC-100, etc.)
- Términos técnicos en inglés de uso común en IT (cloud, compliance, endpoint, firewall, etc.)
- Acrónimos (WAF, CAF, MCSB, SIEM, SOAR, DLP, MFA, PIM, NSG, etc.)
- Contenido dentro de bloques de código
- URLs y rutas de archivo

## Paso 2: Revisión inteligente

Leer el documento completo y revisar:

### Ortografía española
- Tildes faltantes o incorrectas (á, é, í, ó, ú)
- Uso correcto de ñ
- Diéresis (ü) cuando corresponda

### Puntuación
- Signos de apertura: ¿ antes de preguntas, ¡ antes de exclamaciones
- Comas, puntos y punto y coma

### Gramática
- Concordancia de género y número
- Uso correcto de preposiciones
- Verbos conjugados correctamente
- Laísmo, leísmo, loísmo

### Estilo (solo reportar, no corregir automáticamente)
- Oraciones demasiado largas
- Repeticiones innecesarias
- Gerundios mal usados

## Formato de salida

Presentar los resultados en una tabla:

```
| Línea | Error encontrado | Corrección sugerida | Tipo |
|-------|------------------|---------------------|------|
| 11    | certificacion    | certificación       | Tilde |
| 37    | disenar          | diseñar             | Eñe  |
| ...   | ...              | ...                 | ...  |
```

Tipos posibles: Tilde, Eñe, Puntuación, Gramática, Ortografía, Estilo

## Paso 3: Corrección

Después de presentar la tabla, preguntar a Mau:

1. "¿Aplico todas las correcciones?" (corrige todo automáticamente)
2. "¿Quieres revisar una por una?" (muestra cada corrección para aprobar/rechazar)
3. "Solo muéstrame el reporte" (no modifica nada)

Si se elige corregir, aplicar los cambios con Edit sobre el archivo .md. Si existe un .docx con el mismo nombre, regenerarlo con pandoc después de corregir.

## Idiomas soportados

- `es_ES` (español, por defecto)
- Otros idiomas requieren instalar el diccionario hunspell correspondiente

## Prerequisitos

- hunspell instalado: `brew install hunspell`
- Diccionario es_ES en `~/Library/Spelling/` (archivos es_ES.aff y es_ES.dic)
