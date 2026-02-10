---
name: azure-architecture
description: Genera arquitecturas de soluciones Azure siguiendo los estandares de RSM Peru, incluyendo el patron de referencia, seleccion de tiers, estimaciones de costos y checklist de pre-produccion con requisitos ISO 27001. Usar cuando se necesite disenar una nueva solucion en Azure para un cliente de RSM Peru, estimar costos de infraestructura, seleccionar tiers de servicios (dev vs prod), o preparar un ambiente para produccion.
---

# Arquitectura Azure para RSM Peru

## Patron de referencia

Usar este patron como base para soluciones web de clientes RSM Peru. Adaptar segun los requisitos del proyecto.

```
Front Door (CDN + WAF)
  |
Static Web Apps (Angular)
  | HTTPS
App Service (ASP.NET API) + Managed Identity
  |
  +-- Azure SQL
  +-- Redis Cache
  +-- Blob Storage
```

Costo estimado en produccion: ~$310/mes (region East US 2).

## Seleccion de tiers

| Servicio | Dev | Prod | Costo Prod |
|----------|-----|------|------------|
| App Service | B1 | S1 | $75/mes |
| Azure SQL | Basic | S2 | $150/mes |
| Redis Cache | C0 | C1 | $50/mes |
| Blob Storage | LRS | LRS | $5/mes |

Reglas:

- Usar siempre los tiers Dev para ambientes de desarrollo y QA.
- No sobredimensionar: empezar con el tier mas bajo de produccion y escalar si hay evidencia de carga.
- Preferir App Service sobre AKS salvo que el cliente requiera contenedores explicitamente.
- Preferir Azure SQL sobre Cosmos DB salvo que los datos sean NoSQL nativos.

## Checklist de pre-produccion

### Seguridad (requisitos ISO 27001)

- [ ] Managed Identity configurado (no usar connection strings con credenciales)
- [ ] Secrets almacenados en Key Vault
- [ ] HTTPS obligatorio, TLS 1.2 minimo
- [ ] Azure AD con MFA habilitado
- [ ] RBAC configurado con principio de menor privilegio
- [ ] Network Security Groups restrictivos

### Alta disponibilidad

- [ ] Deployment slots configurados (staging/prod)
- [ ] Auto-scaling habilitado con reglas de CPU y memoria
- [ ] Health checks activos en App Service
- [ ] Minimo 2 instancias en produccion
- [ ] Backup automatico de Azure SQL (retencion 35 dias)
- [ ] Geo-redundancia evaluada segun criticidad del cliente

### Monitoreo

- [ ] Application Insights habilitado
- [ ] Alertas configuradas: errores 5xx, latencia > 2s, CPU > 80%
- [ ] Retencion de logs por 7 anos (requisito ISO 27001)
- [ ] Dashboard de Azure Monitor para stakeholders del cliente

### Control de costos

- [ ] Tags obligatorios: cliente, ambiente, responsable, fecha-creacion
- [ ] Budget alerts al 80% y 100% del presupuesto mensual
- [ ] Lifecycle policies en Blob Storage (hot -> cool a 30 dias, cool -> archive a 90 dias)
- [ ] Revisar Azure Advisor semanalmente el primer mes
