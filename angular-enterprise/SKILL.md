---
name: angular-enterprise
description: "Desarrollo Angular empresarial con convenciones de RSM Peru. Usar cuando se creen componentes, servicios, modulos o features de Angular para proyectos de RSM Peru. Tambien cuando se trabaje con formularios reactivos, state management, optimizacion de performance o arquitectura enterprise en proyectos Angular de RSM."
---

# Angular Enterprise - RSM Peru

Convenciones y decisiones de arquitectura especificas de RSM Peru para proyectos Angular empresariales.

## Estructura de proyecto obligatoria

```
src/app/
  core/              # Servicios singleton, guards, interceptors
    services/
    guards/
    interceptors/
  shared/            # Componentes, pipes, directives compartidos
    components/
    pipes/
    directives/
  features/          # Modulos por funcionalidad (lazy-loaded)
    work-items/
    audits/
    reports/
  models/            # Interfaces y types
```

## Convenciones RSM Peru

### Nombres de features del dominio

Los proyectos RSM Peru manejan estas entidades de dominio:
- `work-items`: Elementos de trabajo de auditoria
- `audits`: Procesos de auditoria
- `reports`: Reportes para clientes

### API y servicios

- URL base desde `environment.apiBaseUrl`
- Endpoints siguen el patron `/api/v1/{recurso}`
- Todas las respuestas del backend usan wrapper `ApiResponse<T>` con campo `data`
- Extraer siempre `response.data` con `map()` en el servicio

```typescript
// Patron obligatorio para servicios RSM
private readonly apiUrl = `${environment.apiBaseUrl}/api/v1/workitems`;

getAll(): Observable<WorkItem[]> {
  return this.http.get<ApiResponse<WorkItem[]>>(this.apiUrl).pipe(
    map(response => response.data),
    shareReplay(1),
    catchError(this.handleError)
  );
}
```

### Modelos de dominio

Usar interfaces tipadas con union types para campos de estado:

```typescript
interface WorkItem {
  id: number;
  title: string;
  description: string | null;
  priority: 'Low' | 'Medium' | 'High';
  assignedUserId?: number;
  createdAt: Date;
}
```

### Componentes

- Siempre usar `standalone: true`
- Siempre usar `ChangeDetectionStrategy.OnPush`
- Patron de destruccion con `destroy$` y `takeUntil`, o preferir `async` pipe

### Routing

- Todas las features usan lazy loading con `loadComponent`
- Archivo central: `app.routes.ts`

## Reglas estrictas

- Prohibido usar `any` en TypeScript
- Prohibido `subscribe` dentro de `subscribe` (usar `switchMap`)
- Prohibido omitir `OnPush` en componentes
- Prohibido logica compleja en templates
- Todo observable debe tener estrategia de unsubscribe

## Comandos del proyecto

```bash
# Componente standalone en feature
ng generate component features/{nombre}/{nombre}-detail --standalone

# Servicio en core
ng generate service core/services/{nombre}

# Build produccion
ng build --configuration production
```
