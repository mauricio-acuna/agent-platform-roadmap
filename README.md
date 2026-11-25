# Plataforma de agentes empresariales: vision y roadmap

Este repositorio es el documento de producto y el plan de ejecucion para evolucionar desde un agente concreto de operaciones hasta una plataforma modular que permita **crear, ejecutar, asegurar, evaluar y distribuir agentes de IA**.

> **Tesis del proyecto:** en aplicaciones empresariales, el valor de un agente no se mide por lo que el modelo puede hacer, sino por lo que el sistema permite hacer de forma controlada, recuperable y verificable.

## Punto de partida

El primer caso de uso es [ops-agent-mcp](https://github.com/mauricio-acuna/ops-agent-mcp): un agente de operaciones basado en LangGraph y MCP que consulta pedidos y solicita reembolsos, con aprobacion humana, reglas de negocio, auditoria y pruebas automatizadas.

Ese repositorio es una **aplicacion de referencia**, no el runtime ni la plataforma final. La estrategia es extraer componentes solo cuando un segundo caso real pruebe que la abstraccion es util. No se pretende reescribir todo el agente de una vez.

### Linea base conocida

Al preparar este roadmap, el agente base ejecuta **33 pruebas locales** sin credenciales de proveedor. Incluye controles deterministas, flujo de aprobacion, herramientas locales/MCP, auditoria y observabilidad. Las evaluaciones de trayectoria con modelos reales aun requieren credenciales y resultados publicados; la persistencia durable de aprobaciones y una demo audiovisual tambien estan pendientes. Esta separacion evita confundir capacidades implementadas con trabajo futuro.

## Que producto se quiere construir

Una plataforma para que un desarrollador pueda:

1. Crear un agente a partir de un proyecto inicial versionado.
2. Describir flujos y herramientas con contratos tipados y permisos declarativos.
3. Conectar proveedores de modelos y servidores MCP mediante adaptadores sustituibles.
4. Separar las propuestas del modelo de las decisiones deterministas de autorizacion.
5. Pausar una accion sensible, pedir aprobacion verificable y reanudarla sin duplicar efectos.
6. Probar trayectorias con modelos simulados y reales, y comparar resultados entre versiones.
7. Inspeccionar latencia, coste, errores y decisiones sin almacenar datos sensibles por defecto.
8. Desplegar y mantener agentes con configuracion, versionado, documentacion y compatibilidad explicita.

El producto inicial debe resolver un problema completo y demostrable. La amplitud se incorporara despues de validar contratos, permisos y experiencia de desarrollo.

## Audiencias y problema que resuelve

| Usuario | Necesidad | Resultado esperado |
|---|---|---|
| Desarrollador de aplicaciones | Conectar un modelo a procesos y herramientas sin rehacer controles en cada agente | Runtime, SDK/CLI, contratos y plantillas |
| Responsable de operaciones | Permitir que agentes consulten sistemas y propongan acciones | Ejecuciones trazables, limites y aprobaciones |
| Aprobador humano | Entender exactamente que se ejecutara antes de aceptar | Vista de accion, argumentos, contexto minimo, caducidad y decision |
| Seguridad / plataforma | Revisar permisos, proveedores, datos y cambios | Politicas declarativas, auditoria, evaluaciones y controles CI |
| Mantenedor de skills | Distribuir capacidades sin otorgar permisos implicitos | Paquetes versionados, manifiestos, compatibilidad y pruebas |

## Lo que lo diferencia

- **Control de efectos secundarios:** aprobacion asociada a una accion concreta; no una aprobacion generica para el agente.
- **Seguridad fuera del prompt:** lista de herramientas, validacion, reglas y autorizacion en codigo.
- **Operabilidad:** persistencia, reanudacion, idempotencia, trazas y limites de coste como partes del diseño.
- **Evidencia visible:** pruebas deterministas y evals versionadas; las afirmaciones publicas enlazan resultados reales.
- **Evolucion por casos de uso:** primero pedidos; despues otro dominio que demuestre que el runtime es reutilizable.
- **Adopcion progresiva:** el usuario puede empezar con un agente local y adoptar despues estado durable, CLI o adaptadores.

## Documentacion del proyecto

- [ROADMAP.md](ROADMAP.md) — etapas, entregables, dependencias, gates y estimacion.
- [docs/product-definition.md](docs/product-definition.md) — usuarios, flujos, requisitos y limites del MVP.
- [docs/target-architecture.md](docs/target-architecture.md) — componentes, contratos, flujos de datos y decisiones tecnologicas propuestas.
- [docs/quality-and-security.md](docs/quality-and-security.md) — modelo de amenazas, controles, estrategia de pruebas y gates de release.
- [docs/delivery-plan.md](docs/delivery-plan.md) — backlog inicial priorizado con identificadores, dependencias y criterios comprobables.

## Roadmap resumido

| Etapa | Resultado verificable | Prioridad |
|---|---|---|
| 0. Evidencia del agente base | Demo, pruebas reproducibles, evals etiquetadas y comportamiento de reanudacion especificado | P0 |
| 1. Runtime comun | Dos agentes diferentes ejecutados con los mismos contratos | P0 |
| 2. Politicas y herramientas | Autorizacion central, registro tipado y aprobaciones vinculadas a la accion | P0 |
| 3. Estado durable | Recuperacion, expiracion, aislamiento e idempotencia probados | P0 |
| 4. Evals y observabilidad | Gates deterministas, evals comparables, trazas redactadas y reporte | P0 |
| 5. Skills y plantillas | Paquetes versionados con manifiesto, permisos y pruebas | P1 |
| 6. CLI y adaptadores | Inicializacion y ejecucion reproducible con una primera integracion | P1 |
| 7. Distribucion y mantenimiento | Releases, compatibilidad, guias operativas y proceso de seguridad | P1 |

La prioridad no significa que el trabajo este iniciado. Todos los hitos permanecen pendientes hasta que se adjunte evidencia; el estado detallado se mantiene en [ROADMAP.md](ROADMAP.md).

## No objetivos del MVP

- Entrenar o servir modelos fundacionales propios.
- Convertir el runtime en un sistema autonomo que ejecute cualquier comando.
- Prometer inmunidad contra prompt injection o seguridad perfecta.
- Implementar de entrada un catalogo masivo, decenas de agentes o todos los adaptadores posibles.
- Crear una interfaz web administrativa antes de validar CLI, API y flujos de aprobacion.
- Reemplazar LangGraph antes de tener una razon medida para hacerlo.
- Publicar cifras de calidad, coste o rendimiento que no puedan reproducirse.

## Como se considera exitoso el MVP

El MVP se considera util cuando un desarrollador que no conoce el codigo puede, en un entorno limpio:

- inicializar un agente de solo lectura y otro con accion protegida;
- ejecutar ambos con un proveedor real y un modelo de prueba determinista;
- verificar que una politica deniega una herramienta no autorizada;
- pausar y reanudar una accion sensible sin cambiar los argumentos aprobados ni repetir el efecto;
- correr el conjunto de pruebas y generar un informe de evals;
- inspeccionar una traza sin encontrar prompts, secretos ni argumentos sensibles;
- entender los requisitos, limites y pasos de despliegue desde la documentacion.

Los umbrales de eval se fijaran a partir de una linea base medida; no se inventan en este documento.

## Principios de producto y arquitectura

1. **El modelo propone; el runtime decide.** Las decisiones de permiso y reglas de negocio no se delegan al LLM.
2. **Denegar por defecto.** Una herramienta desconocida, un permiso ambiguo o una politica ausente bloquean la operacion.
3. **Aprobacion ligada a la accion.** La confirmacion caduca y se invalida si cambian herramienta, argumentos, identidad o contexto relevante.
4. **Minimo privilegio y minimizacion de datos.** Cada adaptador recibe solo capacidades y datos necesarios.
5. **Recuperacion segura.** Reintentos, concurrencia, expiracion y fallos parciales se especifican antes de integrar efectos externos.
6. **Observabilidad sin filtraciones.** Metricas y atributos tecnicos por defecto; contenido de usuario solo con justificacion, redaccion y consentimiento.
7. **Abstracciones demostradas.** Una interfaz publica requiere mas de una implementacion o un caso de uso que valide su valor.
8. **Compatibilidad explicitada.** Cada proveedor y adaptador declara que funciones soporta y con que versiones.
9. **Evidencia sobre marketing.** Un gate verde no equivale por si solo a seguridad o calidad de produccion.
10. **Entrega incremental.** Cada etapa debe dejar el sistema en un estado util y verificable.

## Estimacion de planificacion

La estimacion de **12–20 meses** asume una persona con unas 20 horas semanales para el proyecto. Con 12–16 horas semanales, una estimacion mas prudente es **16–35 meses**. Son equivalentes de esfuerzo concentrado, no fechas comprometidas; no incluyen esperas de terceros, cambios de proveedor ni trabajo de comunidad. Puede variar segun dedicacion, infraestructura, integraciones y resultados de evals. Ninguna fase se cierra solo por calendario.

## Contribuir y cambiar el plan

- Antes de abrir trabajo de una etapa posterior, comprobar sus dependencias en [ROADMAP.md](ROADMAP.md).
- Todo cambio de alcance debe incluir motivacion, impacto, alternativas y evidencia esperada.
- Cada tarea de implementacion debe poder mapearse a un requisito o riesgo en la documentacion.
- Registrar decisiones que afecten interfaces, persistencia, permisos o compatibilidad como ADRs futuros.
- No marcar como completado trabajo planificado sin enlaces a pruebas, release o demostracion reproducible.
