---
name: eraser-diagrams
description: Genera diagramas profesionales usando el MCP server de Eraser. Usar cuando se necesite crear diagramas de arquitectura cloud, diagramas de secuencia, ERDs, flowcharts o diagramas BPMN. Soporta Azure, AWS, GCP y diagramas generales. Tambien cuando se pida crear archivos en Eraser con multiples diagramas o exportar PNGs de alta calidad.
---

# Eraser Diagrams

## Parametros por defecto

Siempre usar estos parametros al renderizar PNGs salvo que el usuario pida otra cosa:

```
imageQuality: 3
background: true
colorMode: bold
theme: light
```

## Tipos de diagrama disponibles

| Herramienta | Uso |
|-------------|-----|
| renderCloudArchitectureDiagram | Arquitecturas Azure, AWS, GCP |
| renderSequenceDiagram | Flujos de comunicacion entre servicios |
| renderEntityRelationshipDiagram | Modelos de base de datos |
| renderFlowchart | Flujos de proceso, decisiones |
| renderBpmnDiagram | Procesos de negocio con swimlanes |
| renderPrompt | Cuando no se tiene el DSL, generar con IA desde descripcion |
| renderElements | Multiples diagramas en una sola llamada |
| createFile | Archivo en Eraser con markdown + diagramas embebidos |

## Workflow recomendado

1. **Diagrama simple**: usar render* directamente con DSL
2. **Diagrama desde descripcion**: usar renderPrompt con texto libre
3. **Multiples diagramas**: usar createFile con markdown y bloques embebidos
4. **Exportar PNG**: siempre con imageQuality 3 y background true
5. **Edicion fina**: generar el DSL aca, pegar en Eraser para ajustar layout manualmente

## Limites

- Diagramas con ~20 nodos funcionan bien
- Mas de ~30 nodos con muchas conexiones pueden dar HTTP 503
- Si falla: simplificar nombres, reducir conexiones, quitar niveles de anidamiento
- La API no permite listar ni leer archivos existentes

## Sintaxis DSL para arquitectura cloud

```
// Nodos
Nombre [icon: azure-app-service]
Nombre [icon: azure-app-service, color: blue]

// Grupos (anidables)
Azure [icon: azure] {
  VNet [icon: azure-virtual-network] {
    Subnet {
      VM [icon: azure-vm]
    }
  }
}

// Conexiones
Origen > Destino
Origen > Destino: Etiqueta
Origen <> Destino: Bidireccional
```

## Iconos Azure mas usados

### Compute y Web
azure-app-service, azure-kubernetes-service, azure-vm, azure-functions

### Networking
azure-virtual-network, azure-firewall, azure-front-door, azure-application-gateway,
azure-load-balancers, azure-vpn-gateway, azure-expressroute-circuits,
azure-virtual-network-gateways, azure-bastion, azure-web-application-firewall,
azure-dns, azure-private-link, azure-network-watcher, azure-ddos-protection

### Data
azure-sql-database, azure-cosmos-db, azure-cache-redis, azure-blob-storage,
azure-file-storage, azure-storage

### Seguridad e Identidad
azure-active-directory, azure-key-vault, azure-managed-identities,
azure-security-center

### Mensajeria
azure-service-bus, azure-event-grid, azure-event-hubs

### DevOps y Monitoring
azure-devops, azure-container-registries, azure-application-insights,
azure-log-analytics, azure-monitor, azure-action-groups

### Generales
azure, azure-resource-groups

## Iconos generales
users, mobile, server, router, globe, monitor, database, cloud, lock, tool

## Sintaxis para otros tipos de diagrama

### Secuencia
```
title Flujo de Autenticacion
Client [icon: monitor]
Server [icon: server]
Client > Server: Login request
Server --> Client: JWT token
```

### ERD
```
users [icon: user, color: blue] {
  id int pk
  email string
  name string
}
orders [icon: shopping-cart] {
  id int pk
  user_id int
}
users.id < orders.user_id
```

### Flowchart
```
title Mi Flujo
direction right
Start [shape: oval]
Decision [shape: diamond]
Start > Decision
Decision > Action A: Yes
Decision > Action B: No
```

### BPMN
```
title Proceso
Pool A [color: blue] {
  Task 1 [icon: edit]
  Task 2 [icon: check]
}
Pool B [color: green] {
  Task 3 [icon: tool]
}
Task 1 > Task 2
Task 2 --> Task 3: Message flow
```

## API REST complementaria

Para operaciones que el MCP no cubre, usar curl con el token de `~/.claude.json`:

```bash
# Metricas de uso
curl -H "Authorization: Bearer $TOKEN" "https://app.eraser.io/api/reports/usage"

# Recuperar DSL de un request previo
curl -H "Authorization: Bearer $TOKEN" "https://app.eraser.io/api/ai-requests/{requestId}"
```

## Ubicacion de archivos

Los PNGs generados deben guardarse en `documentacion/diagramas/` dentro del proyecto correspondiente. Los archivos .drawio tambien van en esa carpeta.

## Convenciones Azure RSM Peru

Cuando el diagrama sea para un cliente de RSM Peru:
- Seguir el patron de referencia: Front Door > App Service > Azure SQL + Redis + Blob
- Incluir Key Vault y Managed Identity
- Incluir Application Insights y Log Analytics
- Usar nombres de servicios en ingles (estandar Azure)
- Titulo del diagrama debe incluir el nombre del cliente o proyecto
