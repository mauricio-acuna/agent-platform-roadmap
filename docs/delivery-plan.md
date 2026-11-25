# Plan de entrega y backlog inicial

## Como leer este backlog

- Los IDs `PLAT-NNN` son referencias estables para issues y pull requests.
- `P0` bloquea una plataforma que pueda realizar acciones externas; `P1` mejora adopcion y distribucion; `P2` se pospone hasta validar necesidad.
- Una tarea se divide si supera varios dias de trabajo o mezcla mas de una frontera de seguridad.
- **Estado inicial de todas las tareas: pendiente.** No iniciar una tarea si su dependencia no esta cerrada o si falta una decision necesaria.

## Fase A — Baseline verificable

**Objetivo:** que el agente actual sea una referencia de comportamiento y seguridad antes de extraer abstracciones.
**Salida:** informe de pruebas/evals, demo y especificacion de recuperacion.
**Riesgo principal:** confundir test determinista con comportamiento del modelo real.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-001 | P0 | Ejecutar CI y fijar baseline de pruebas deterministas | Ninguna | Enlace al run y commit; fallos triageados, no silenciados |
| PLAT-002 | P0 | Versionar corpus de evals y metadatos de ejecucion | PLAT-001 | Cada caso tiene ID, categoria, severidad y expected/forbidden actions |
| PLAT-003 | P0 | Correr primera eval real en un modelo configurado | PLAT-002 | Informe identifica modelo/config y separa fallos criticos |
| PLAT-004 | P0 | Documentar estado actual de ejecuciones y aprobaciones | PLAT-001 | Tabla de estados, transiciones y comportamientos ambiguos acordados |
| PLAT-005 | P1 | Grabar demo reproducible de aprobacion y rechazo | PLAT-001, PLAT-004 | Instrucciones y video/GIF no exponen claves ni datos reales |
| PLAT-006 | P1 | Definir datos que pueden persistir y politica de retencion | PLAT-004 | Matriz revisada en product-definition y quality-security |

## Fase B — Persistencia y recuperacion segura

**Objetivo:** poder pausar/reanudar procesos y recuperarse de errores sin repetir efectos.
**Salida:** adapter de store durable, test suite de restart/replay y migracion inicial.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-010 | P0 | Definir maquina de estados de ejecucion y aprobacion | PLAT-004 | Estados terminales, expiracion y error ambiguo explicitados |
| PLAT-011 | P0 | Definir contrato `StateStore` y fake de pruebas | PLAT-010 | Runtime no depende de implementacion concreta del almacen |
| PLAT-012 | P0 | Implementar store local durable | PLAT-011, PLAT-006 | Reinicio restaura ejecucion; tests no requieren proveedor |
| PLAT-013 | P0 | Persistir `ApprovalRequest` con payload digest | PLAT-010, PLAT-012 | Mutacion de cualquier campo invalida la aprobacion |
| PLAT-014 | P0 | Anadir claves idempotentes a operaciones con efectos | PLAT-010 | Repeticion y concurrencia no duplican accion en el adapter de prueba |
| PLAT-015 | P0 | Implementar recuperacion `outcome_unknown` | PLAT-014 | No se reintenta a ciegas; estado queda reconciliable |
| PLAT-016 | P1 | Definir store remoto y migracion versionada | PLAT-012, PLAT-006 | Migracion ida/recuperacion probadas con fixtures y sin perder estado |
| PLAT-017 | P1 | Probar aislamiento por sujeto/tenant | PLAT-012, PLAT-013 | Pruebas cruzadas deniegan lectura, aprobacion y reanudacion |

## Fase C — Runtime y contratos compartidos

**Objetivo:** probar que el nucleo sirve para mas de un problema sin acoplarse al dominio de pedidos.
**Salida:** runtime reusable y dos agentes funcionales.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-020 | P0 | Extraer tipo `AgentDefinition` y validacion de config | PLAT-001, PLAT-010 | Config invalida falla al arrancar con error accionable |
| PLAT-021 | P0 | Definir interfaces de modelo, executor y approval handler | PLAT-020 | Tipos y errores cubiertos por contract tests |
| PLAT-022 | P0 | Mover el agente de pedidos al runtime sin cambio funcional | PLAT-021, PLAT-012 | Evals y pruebas existentes conservan comportamiento |
| PLAT-023 | P0 | Crear segundo agente read-only de otro dominio | PLAT-022 | Reutiliza runtime y politicas sin copiar logica transversal |
| PLAT-024 | P1 | Crear adapter a segundo proveedor real | PLAT-021, PLAT-003 | Capacidades y diferencias se declaran, pruebas con fixture |
| PLAT-025 | P1 | Publicar paquete interno inicial y guia de consumo | PLAT-022, PLAT-023 | Proyecto externo de ejemplo instala version etiquetada |

## Fase D — Registro MCP y politicas

**Objetivo:** establecer un plano de control predecible para herramientas, riesgo y permisos.
**Salida:** registro validado, politica central, audit trail y pruebas de abuso.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-030 | P0 | Definir schema de manifiesto de herramienta | PLAT-021 | Requiere id/version/input/output/capability/risk/timeout |
| PLAT-031 | P0 | Implementar carga y validacion de manifiestos | PLAT-030 | Rechaza campos desconocidos peligrosos y esquemas invalidos |
| PLAT-032 | P0 | Implementar `PolicyDecision` y decision deny-by-default | PLAT-030 | Herramienta desconocida o politica ausente no se ejecuta |
| PLAT-033 | P0 | Vincular aprobacion con sujeto, politica y digest de argumentos | PLAT-013, PLAT-032 | Cambiar sujeto, args o policy version invalida decision |
| PLAT-034 | P0 | Validar servidor MCP y sincronizacion de herramientas | PLAT-030 | Servidor extra o metadata alterada no amplian allowlist |
| PLAT-035 | P0 | Implementar timeout, limite de respuesta y errores tipados | PLAT-030 | Timeout y payload demasiado grande fallan sin success-shaped fallback |
| PLAT-036 | P1 | Definir permisos por rol/identidad y scope | PLAT-017, PLAT-032 | Acceso least-privilege verificado en casos positivos y negativos |
| PLAT-037 | P1 | Crear CLI para inspeccionar permisos efectivos | PLAT-031, PLAT-032 | Muestra decision y origen de regla sin exponer secretos |

## Fase E — Evals y observabilidad de plataforma

**Objetivo:** convertir pruebas propias del agente en capacidades reusables para varios agentes.
**Salida:** herramienta de eval, artefactos comparables, trazas sin contenido sensible y gates.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-040 | P0 | Definir formato de dataset versionado | PLAT-002, PLAT-023 | Schemas validan input, fixtures, expected/forbidden y severidad |
| PLAT-041 | P0 | Extraer runner de trayectorias independiente del agente | PLAT-040 | Ejecuta tests de ambos agentes con el mismo formato |
| PLAT-042 | P0 | Agregar comparacion de baseline y reporte JSON/Markdown | PLAT-041 | Reporte liga commit, config, modelo y resultados por categoria |
| PLAT-043 | P0 | Establecer gates de casos criticos | PLAT-042, PLAT-032 | Cualquier bypass critico bloquea release independientemente del promedio |
| PLAT-044 | P0 | Estandarizar atributos OpenTelemetry permitidos | PLAT-021 | Lista allowlisted; no incluye prompts, args o secretos |
| PLAT-045 | P0 | Agregar tests canario para detectar filtraciones de telemetria | PLAT-044 | Prueba recorre exporters y falla si detecta un valor secreto |
| PLAT-046 | P1 | Medir coste/latencia por proveedor y herramienta | PLAT-044 | Reporte percentiles, errores y datos de volumen |
| PLAT-047 | P1 | Generar dashboard de ejemplo | PLAT-046 | Dashboard usa datos sinteticos y documenta metricas |

## Fase F — Skills y templates

**Objetivo:** habilitar extension por capacidades reutilizables sin convertir instrucciones en permisos implicitos.
**Salida:** formato de paquete y catalogo inicial pequeno.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-050 | P1 | Definir manifiesto de skill y compatibilidad | PLAT-030, PLAT-040 | Versionado, requisitos, pruebas y permisos declarados |
| PLAT-051 | P1 | Implementar validador de paquetes | PLAT-050 | Paquete incompatible o permiso no autorizado se rechaza |
| PLAT-052 | P1 | Crear template de agente read-only | PLAT-023, PLAT-050 | Se genera y ejecuta offline con tests incluidos |
| PLAT-053 | P1 | Crear template de accion con aprobacion | PLAT-033, PLAT-050 | Aprobacion, rechazo y replay tests incluidos |
| PLAT-054 | P1 | Definir procedencia y revision de paquetes | PLAT-051 | Version publica enlaza fuente, changelog y pruebas |
| PLAT-055 | P2 | Publicar catalogo inicial | PLAT-052, PLAT-053, PLAT-054 | Solo paquetes revisados; proceso de retiro documentado |

## Fase G — CLI e integraciones

**Objetivo:** reducir friccion de adopcion y probar una integracion real antes de multiplicar adaptadores.
**Salida:** CLI estable y un adaptador probado de extremo a extremo.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-060 | P1 | Definir comandos y experiencia de configuracion | PLAT-020, PLAT-052 | `init`, `validate`, `run`, `eval` tienen ayuda y codigos de salida |
| PLAT-061 | P1 | Implementar generacion desde plantilla versionada | PLAT-060, PLAT-051 | No sobrescribe archivos sin confirmacion; lista permisos |
| PLAT-062 | P1 | Probar instalacion limpia en Windows/Linux | PLAT-061 | CI ejecuta smoke test en matrices soportadas |
| PLAT-063 | P1 | Elegir primer adaptador de IDE/agente de desarrollo | PLAT-060 | ADR basado en usuarios objetivo y capacidades requeridas |
| PLAT-064 | P1 | Implementar adaptador y matriz de capacidades | PLAT-063, PLAT-032 | No reimplementa politicas; instala, ejecuta y desinstala |
| PLAT-065 | P2 | Evaluar adaptador adicional | PLAT-064 | Requiere feedback/uso y contrato de conformidad |

## Fase H — Release y operacion

**Objetivo:** que el producto pueda versionarse, actualizarse, soportarse y retirar componentes.
**Salida:** release publica con guias de operacion, contribucion y seguridad.

| ID | Prioridad | Tarea | Dependencias | Criterio de aceptacion |
|---|---|---|---|---|
| PLAT-070 | P1 | Definir versionado semantico y politica de soporte | PLAT-025 | Compatibilidad y fin de soporte publicados |
| PLAT-071 | P1 | Generar SBOM, escanear dependencias y firmar artefactos segun necesidad | PLAT-070 | Release asociada a commit, workflow y dependencias |
| PLAT-072 | P1 | Publicar threat model y procedimiento de incidente | PLAT-033, PLAT-045 | Procedimientos para bloquear acciones, revocar aprobaciones y preservar evidencia |
| PLAT-073 | P1 | Probar upgrade y rollback desde versiones anteriores | PLAT-070 | Smoke automatizado y migracion recuperable |
| PLAT-074 | P1 | Publicar guia de contribucion y disclosure responsable | PLAT-072 | Canal y SLA inicial descritos sin prometer respuesta irreal |
| PLAT-075 | P1 | Crear release candidate y obtener feedback de usuarios | PLAT-071, PLAT-073, PLAT-074 | Issues de feedback clasificadas y decisiones registradas |

## Dependencias principales

```text
PLAT-001 -> PLAT-002 -> PLAT-003
PLAT-004 -> PLAT-010 -> PLAT-011 -> PLAT-012 -> PLAT-013 -> PLAT-014/015
PLAT-020 -> PLAT-021 -> PLAT-022 -> PLAT-023
PLAT-030 -> PLAT-031 -> PLAT-032 -> PLAT-033/034/035
PLAT-040 -> PLAT-041 -> PLAT-042 -> PLAT-043
PLAT-044 -> PLAT-045 -> PLAT-072
PLAT-030 + PLAT-040 -> PLAT-050 -> PLAT-051 -> PLAT-052/053
PLAT-020 + PLAT-052 -> PLAT-060 -> PLAT-061 -> PLAT-062
PLAT-060 + PLAT-032 -> PLAT-063 -> PLAT-064
PLAT-025 + PLAT-064 + PLAT-072 -> PLAT-070...075
```

## Trazabilidad de requisitos

La tabla enlaza los requisitos funcionales definidos en [product-definition.md](product-definition.md) con los paquetes de trabajo que los hacen verificables. El detalle de cobertura se completa en el PR de implementacion al añadir pruebas concretas.

| Requisitos | Tareas principales | Evidencia esperada |
|---|---|---|
| FR-001, FR-002 | PLAT-020–025 | Dos agentes comparten runtime; un fake sustituye el modelo en pruebas |
| FR-003, FR-004, FR-005, FR-014 | PLAT-030–037, PLAT-063–065 | Manifiestos versionados, contratos validados, permisos y matriz de capacidades |
| FR-006, FR-007, FR-008, FR-009 | PLAT-010, PLAT-013, PLAT-032–034 | Regla de dominio externa al LLM; approval digest e invariantes de revalidacion |
| FR-010, FR-011 | PLAT-010–017 | Restart/replay, idempotencia, expiracion y recuperacion probados |
| FR-012, FR-013 | PLAT-012–017, PLAT-044–045 | Eventos de auditoria y pruebas canario contra fuga de datos |
| FR-015, FR-016 | PLAT-040–047 | Metricas trazables, comparacion de baseline e informe de eval reproducible |
| FR-017, FR-018 | PLAT-050–055, PLAT-060–061 | Templates y skills versionados, validados y con permisos explicitos |
| FR-019 | PLAT-006, PLAT-012, PLAT-016–017 | Retencion, aislamiento y borrado verificables |
| FR-020 | No comprometido para MVP | Requiere validar necesidad de operador antes de iniciar una UI |

FR-020 se mantiene expresamente fuera del compromiso inicial. No es un hueco accidental: la API y CLI deben validar primero el modelo de aprobacion y soporte operativo.

## Primera iteracion recomendada

No iniciar todas las tareas a la vez. Primer batch:

1. PLAT-001: confirmar estado real de pruebas y CI.
2. PLAT-002: preparar formato y corpus inicial de evals.
3. PLAT-004: definir maquina de estados de ejecucion/aprobacion.
4. PLAT-006: acordar politica de datos y retencion.
5. PLAT-010/011: disenar transiciones y contrato del store.

La implementacion de persistencia empieza solo despues de acordar estados, reintentos, expiracion e idempotencia. De lo contrario se persiste un comportamiento ambiguo que luego resulta caro de corregir.

## Definicion de listo para empezar

Una tarea puede entrar en progreso cuando:

- tiene un problema y usuario afectado definidos;
- identifica el requisito o amenaza relacionada;
- tiene dependencias desbloqueadas y decisiones necesarias registradas;
- especifica una prueba que demostrara el resultado;
- su alcance cabe en un PR revisable y no mezcla cambios no relacionados.

Si una tarea cambia permisos, identidad, almacenamiento o exportacion de datos, incluir desde el inicio pruebas negativas y revision de seguridad.

## Estimacion y estrategia de capacidad

La estimacion agregada es de 27–45 semanas-persona. Con una dedicacion aproximada de 20 horas semanales, eso equivale a 12–20 meses; con 12–16 horas semanales, puede equivaler a 16–35 meses. Incluye implementacion, aprendizaje, documentacion y evaluacion; no presupone ocho proyectos independientes ni desarrollo paralelo. Actualizarla con datos reales al completar cada fase.

Orden de prioridad ante falta de tiempo:

1. Asegurar y demostrar el flujo de accion sensible.
2. Persistencia e idempotencia.
3. Segundo caso que valide el runtime.
4. Evals y telemetria confiables.
5. CLI y primer adaptador.
6. Skills, catalogo y distribucion ampliada.

## Reglas para cerrar una tarea

- El cambio tiene objetivo, alcance y criterio de aceptacion antes de implementarse.
- Pruebas nuevas cubren el riesgo o comportamiento introducido.
- Pruebas relevantes existentes siguen activas.
- Documentacion y ejemplos reflejan la implementacion real.
- Errores, limites y compatibilidad se comunican explicitamente.
- El commit/PR enlaza el ID `PLAT-NNN` y aporta evidencia.
