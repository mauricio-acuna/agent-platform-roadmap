# Ruta de aprendizaje: diseno de una plataforma de agentes

## Proposito

Este repositorio ensena a razonar sobre la evolucion de una aplicacion de agente hacia una plataforma reusable. Es un documento de producto, arquitectura propuesta y plan de trabajo; no es codigo ejecutable ni prueba que las capacidades descritas ya existan.

La ruta esta destinada a estudiantes, docentes y equipos que quieran estudiar conceptos, leer evidencia y tomar decisiones de diseno con rigor. El caso implementado de partida es [ops-agent-mcp](https://github.com/mauricio-acuna/ops-agent-mcp); cada afirmacion acerca de su estado debe verificarse en ese repositorio y en la tabla de estado de [ROADMAP.md](../ROADMAP.md).

## Resultados de aprendizaje

Al finalizar la ruta, el estudiante deberia poder:

1. Distinguir vision de producto, arquitectura objetivo, backlog y evidencia de implementacion.
2. Derivar requisitos verificables desde necesidades de usuarios y amenazas.
3. Explicar por que los permisos, las reglas de negocio y las aprobaciones no deben depender de una respuesta del modelo.
4. Diseñar contratos entre runtime, proveedor, herramientas, politica, aprobacion y almacenamiento.
5. Modelar persistencia, recuperacion, idempotencia y fallos ambiguos.
6. Elegir pruebas unitarias, de contrato, integracion, trayectoria y despliegue segun la afirmacion que se quiera verificar.
7. Evaluar privacidad, observabilidad, compatibilidad y riesgo operacional como propiedades del sistema.
8. Presentar decisiones y limites con evidencia y separar claramente lo implementado de lo propuesto.

## Prerrequisitos sugeridos

- Programacion basica y lectura de diagramas de componentes.
- Familiaridad inicial con HTTP, JSON, APIs y pruebas automatizadas.
- No se presupone experiencia previa con LLMs, agentes, MCP, seguridad de aplicaciones o almacenamiento durable; estos temas se introducen aqui y se complementan con la guia practica del repositorio de referencia.

## Como recorrer los documentos

| Orden | Documento | Uso didactico |
|---|---|---|
| 1 | [README.md](../README.md) | Vision, alcance, usuarios y estado resumido |
| 2 | [docs/product-definition.md](product-definition.md) | Problemas, actores, casos de uso y requisitos |
| 3 | [docs/quality-and-security.md](quality-and-security.md) | Activos, amenazas, invariantes, pruebas y evaluacion |
| 4 | [docs/target-architecture.md](target-architecture.md) | Fronteras, contratos, flujos y recuperacion |
| 5 | [ROADMAP.md](../ROADMAP.md) | Secuencia de hitos, gates y estado con evidencia |
| 6 | [docs/delivery-plan.md](delivery-plan.md) | Tareas trazables, dependencias y criterios de aceptacion |
| 7 | [ops-agent-mcp: guia de aprendizaje](https://github.com/mauricio-acuna/ops-agent-mcp/blob/main/docs/learning-guide.md) | Recorrido practico por una implementacion concreta |

El orden importa: primero se entiende el problema; despues las amenazas y arquitectura; finalmente se examina el trabajo planificado. Leer una propuesta como si ya estuviera implementada es un error de interpretacion.

## Unidades de aprendizaje

### Unidad 1 — De una aplicacion a un producto de plataforma

**Conceptos:** problema de usuario, caso de uso, MVP, no objetivo, criterio de exito, evidencia.

**Lecturas:** README y secciones 1–8 de `product-definition.md`.

**Idea clave:** una plataforma no es una coleccion de abstracciones. Debe resolver una necesidad recurrente mejor que copiar una aplicacion concreta, y el segundo caso de uso debe demostrar que la reutilizacion es genuina.

**Actividad:** redactar una necesidad para desarrollador, operador y aprobador; convertir cada una en un resultado observable. Distinguir una capacidad P0, P1 y P2 sin confundir prioridad con estado.

### Unidad 2 — Requisitos y trazabilidad

**Conceptos:** requisito funcional, requisito no funcional, identificador estable, dependencia, criterio de aceptacion y trazabilidad.

**Lecturas:** requisitos `FR-NNN` en `product-definition.md` y tareas `PLAT-NNN` en `delivery-plan.md`.

**Idea clave:** “soportar seguridad” no es verificable. Un criterio util especifica condicion, comportamiento esperado y evidencia, por ejemplo bloquear una herramienta no registrada antes de invocar el ejecutor.

**Actividad:** elegir un requisito y mapearlo a una amenaza, una tarea, una prueba y un artefacto. Si falta un enlace, anotar que evidencia seria necesaria.

### Unidad 3 — Modelo de amenazas y defensa en profundidad

**Conceptos:** activo, adversario, frontera de confianza, amenaza, control preventivo, deteccion, riesgo residual e invariante.

**Lecturas:** `quality-and-security.md`, secciones 2–5 y 8–10; fronteras de confianza en `target-architecture.md`.

**Idea clave:** no se intenta convencer al modelo de ser seguro. Se limita lo que el sistema permite hacer, incluso si una propuesta del modelo es maliciosa o incorrecta. Ningun control aislado prueba seguridad total.

**Actividad:** para una herramienta de escritura, dibujar el flujo desde la propuesta hasta el efecto externo. Marcar en que fronteras se validan argumentos, identidad, aprobacion, expiracion y resultado.

### Unidad 4 — Contratos y desacoplamiento

**Conceptos:** contrato, interfaz, adaptador, proveedor, executor, manifiesto, compatibilidad y abstraccion prematura.

**Lecturas:** seccion 7 de `target-architecture.md`; fase C de `delivery-plan.md`; `AgentDefinition` en el repositorio de referencia.

**Idea clave:** una interfaz establece el comportamiento que un consumidor puede esperar. No basta con extraer un protocolo: hace falta probarlo con implementaciones y casos consumidores. Los adaptadores pueden tener capacidades diferentes; no se debe fingir paridad.

**Actividad:** definir un contrato minimo para proveedor de modelos. Especificar entradas, salidas, errores y capacidades no soportadas. Proponer un test de contrato que pueda ejecutar un fake.

### Unidad 5 — Estado, aprobaciones e idempotencia

**Conceptos:** maquina de estados, checkpoint, aprobacion ligada al payload, expiracion, concurrencia, idempotencia y resultado ambiguo.

**Lecturas:** secciones 3–6 de `target-architecture.md`, caso UC-03/UC-04 en `product-definition.md` y fase B de `delivery-plan.md`.

**Idea clave:** un timeout no demuestra que la operacion no ocurrio. Si el destino puede haber aplicado un efecto sin que llegue la respuesta, el sistema debe reconciliar el estado en vez de reintentar a ciegas.

**Actividad:** modelar reinicio del proceso despues de aprobar y antes de recibir respuesta externa. Definir estados, transiciones invalidas y pruebas de repeticion/concurrencia.

### Unidad 6 — Pruebas y evaluacion de agentes

**Conceptos:** test unitario, test de contrato, integracion, eval de trayectoria, dataset versionado, metricas criticas y gate de release.

**Lecturas:** secciones 6–7 y 9 de `quality-and-security.md`; la suite y `evals/` del repositorio de referencia.

**Idea clave:** los tests deterministas verifican invariantes especificos; las evals reales miden comportamiento de modelos en escenarios y pueden variar. Una tasa promedio no debe ocultar un fallo critico de autorizacion.

**Actividad:** clasificar cinco afirmaciones del proyecto segun el tipo de prueba que las respaldaria. Identificar que no demuestra cada resultado.

### Unidad 7 — Datos, auditoria y observabilidad

**Conceptos:** minimizacion de datos, clasificacion, retencion, auditoria, trazas, metricas, redaccion y correlacion.

**Lecturas:** secciones 8 de `target-architecture.md` y `quality-and-security.md`.

**Idea clave:** auditoria registra hechos relevantes para explicar decisiones; telemetria mide el comportamiento operacional. No requieren el mismo detalle ni retencion. Un identificador de ejecucion puede ser util sin guardar el prompt completo.

**Actividad:** disenar un esquema de evento que permita investigar una accion sin incluir secreto, prompt, respuesta completa o payload financiero.

### Unidad 8 — Planificacion, gates y operacion

**Conceptos:** dependencia, gate, criterio de salida, riesgo residual, rollback, versionado y release.

**Lecturas:** `ROADMAP.md`, `delivery-plan.md` y seccion 9 de `quality-and-security.md`.

**Idea clave:** terminar una tarea no equivale a cerrar un hito. Cada gate exige artefacto, prueba, operacion, revision de seguridad y compatibilidad. Un cambio futuro debe mantener visible lo que aun no se ha verificado.

**Actividad final:** preparar una revision de hito con evidencias, fallos conocidos, riesgos residuales y recomendacion explicita de avanzar o bloquear.

## Proyecto final propuesto

Diseñar un segundo agente de solo lectura para un dominio ficticio distinto de pedidos. El resultado academico debe incluir:

1. Caso de uso, actores, alcance y no objetivos.
2. Definicion de agente y manifiesto de herramientas.
3. Modelo de amenazas y fronteras de confianza.
4. Contratos y reglas de negocio separadas del prompt.
5. Pruebas unitarias y de contrato sin credenciales.
6. Eval de trayectoria propuesta, con casos prohibidos y criterio critico.
7. Esquema de auditoria y politica de datos.
8. Guia reproducible que identifique dependencias, configuracion y limitaciones.

No es necesario implementar persistencia, UI ni proveedor real para aprobar el diseno. Si se implementa una capacidad, debe adjuntarse evidencia reproducible y describir fallos conocidos.

## Plantilla minima para una leccion o concepto

Cada material de curso derivado de estos repositorios debe incluir:

1. **Pregunta de aprendizaje:** que problema resolvera el concepto.
2. **Definicion y modelo mental:** explicacion inicial en lenguaje claro, con terminos tecnicos definidos.
3. **Motivacion:** que falla o riesgo aparece si no se usa.
4. **Ejemplo concreto:** flujo o codigo del repositorio con ruta y simbolo.
5. **Limites:** que no resuelve, que supuestos tiene y que no se ha implementado.
6. **Practica:** tarea con resultado esperado y forma de verificarlo.
7. **Reflexion:** trade-offs, alternativas o preguntas de diseno.
8. **Fuentes:** documentacion primaria y fecha/version cuando el concepto dependa de tecnologia cambiante.

Las afirmaciones de seguridad, compatibilidad y estado deben citar pruebas o documentacion primaria; evitar presentar una prueba local como certificacion formal.

## Vocabulario esencial

- **Runtime:** componente que coordina ciclo de vida, limites, estado y ejecucion de un agente.
- **Policy engine:** componente que decide deterministamente si una accion se permite, deniega o requiere aprobacion.
- **Tool registry:** fuente local de contratos, esquemas, versiones y capacidades de herramientas.
- **Approval request:** solicitud vinculada a una accion concreta que un aprobador revisa antes de ejecutar.
- **State store:** interfaz/implementacion que conserva ejecuciones y transiciones para consulta o recuperacion.
- **Checkpoint:** representacion suficiente del estado para continuar un flujo; persistirlo no garantiza por si solo ejecucion segura.
- **Idempotency key:** identificador usado para que repetir una solicitud no duplique el efecto cuando el destino coopera.
- **Outcome unknown:** resultado no confirmado donde la accion podria haberse aplicado; requiere reconciliacion.
- **Evals:** conjunto versionado de escenarios y metricas para observar comportamiento del agente.
- **MCP:** protocolo de comunicacion para descubrir e invocar herramientas; no constituye autorizacion por si mismo.
- **ADR:** registro de una decision de arquitectura, su contexto, alternativas y consecuencias.

## Convenciones editoriales para ambos repositorios

- Mantener cada explicacion cerca de la fuente de verdad: la implementacion concreta en `ops-agent-mcp`; las decisiones, secuencia y futuro en este roadmap.
- Etiquetar el contenido como **implementado**, **parcial**, **propuesto** o **pendiente** cuando pueda confundirse el estado.
- Enlazar concepto → codigo/diagrama → prueba/evidencia → limitacion.
- Definir acronimos en su primera aparicion y mantener un glosario comun.
- Mostrar tanto el camino exitoso como un fallo relevante y su recuperacion esperada.
- Actualizar guias educativas en el mismo cambio que altera un concepto, contrato o garantia descrita.
- Preferir documentacion oficial del protocolo/framework y citar versiones o fechas para especificaciones cambiantes.
- No copiar secretos, datos reales, prompts privados ni trazas con informacion sensible a ejemplos didacticos.
