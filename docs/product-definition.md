# Definicion de producto

## 1. Resumen ejecutivo

Se propone una plataforma modular para aplicaciones que usan modelos de lenguaje como parte de flujos de trabajo con herramientas. La plataforma no intenta hacer que el modelo sea infalible; hace que las capacidades del modelo esten rodeadas por contratos, permisos, aprobaciones, persistencia, evaluacion y operacion.

El primer producto de referencia es un agente de operaciones de pedidos. Su alcance acotado hace visibles riesgos reales: datos no confiables, seleccion de herramienta, acciones economicas, aprobacion humana, reintentos y auditoria. Una vez que el runtime se use tambien en un segundo dominio, se extraeran sus componentes comunes.

## 2. Vision y resultado de usuario

**Vision:** que un equipo de desarrollo pueda entregar agentes con acciones externas usando patrones repetibles de autorizacion, evaluacion y operacion, sin implementar de nuevo esos controles para cada caso.

**Resultado principal para el desarrollador:** pasar de una idea de agente a una aplicacion comprobable y operable, sabiendo que herramientas puede invocar, por que una accion se permitio, como se recupera una ejecucion y que evidencia respalda una version.

## 3. Usuarios y responsabilidades

### Autor del agente

- Define el caso de uso, datos, herramientas, politica, respuesta esperada y metricas.
- Puede proponer una configuracion, pero no conceder permisos indirectamente mediante texto del prompt.
- Es responsable de agregar ejemplos y evals del dominio.

### Operador del sistema

- Configura proveedores, almacenes, limites y conectores.
- Gestiona ejecuciones, fallos, versiones y retencion de datos.
- No puede modificar la politica de una ejecucion ya aprobada sin invalidar la aprobacion.

### Aprobador de una accion

- Recibe una representacion legible y fiel de la accion que se ejecutara.
- Puede aceptar o rechazar antes de la expiracion.
- No debe aprobar una accion cuyo contenido cambie despues de su decision.

### Revisor de seguridad

- Revisa amenazas, permisos, datos almacenados, integraciones y comportamiento ante abuso.
- Puede bloquear la publicacion cuando haya una regresion critica.
- Necesita evidencia tecnica, no una declaracion del modelo.

## 4. Casos de uso

### UC-01: crear agente desde plantilla

Un desarrollador elige un tipo de agente, establece proveedor, datos, herramientas y politica; genera un proyecto con pruebas locales y una guia de ejecucion.

**Resultado observable:** el proyecto arranca en modo de prueba sin claves externas y muestra con claridad los permisos requeridos.

### UC-02: responder con datos de solo lectura

El modelo interpreta una pregunta y propone una herramienta permitida. El runtime valida argumentos, ejecuta la lectura y devuelve una respuesta con el nivel de evidencia disponible.

**Restriccion:** el contenido devuelto por la herramienta se considera dato no confiable, no instruccion de control.

### UC-03: solicitar una accion sensible

El agente prepara una accion, el runtime valida negocio y politica, crea una solicitud de aprobacion ligada a los parametros y pausa la ejecucion. Un aprobador acepta o rechaza. Si acepta antes de que expire, el runtime verifica de nuevo la politica y ejecuta con una clave idempotente.

**Resultado observable:** historial de decision y ejecucion con referencias, no necesariamente con el payload sensible completo.

### UC-04: recuperarse de un fallo

Tras un reinicio, timeout o respuesta ambigua de un servicio externo, el operador inspecciona el estado. El agente no vuelve a ejecutar a ciegas una accion con efecto lateral.

### UC-05: validar una nueva version del agente

El equipo ejecuta pruebas deterministas y evals con LLM sobre un dataset versionado, compara resultados y decide si la nueva version puede promocionarse.

### UC-06: distribuir una capacidad reutilizable

Un mantenedor publica una skill/plantilla con manifiesto, version, requisitos, permisos y pruebas. Un agente consumidor no obtiene permisos nuevos por instalarla automaticamente.

## 5. Requisitos funcionales iniciales

Los identificadores permiten enlazar requisitos con issues, implementacion y pruebas.

| ID | Requisito | Prioridad |
|---|---|---|
| FR-001 | Ejecutar un agente mediante una definicion y configuracion validadas | P0 |
| FR-002 | Sustituir el modelo por un fake determinista en pruebas | P0 |
| FR-003 | Exponer herramientas mediante contratos tipados y versionados | P0 |
| FR-004 | Denegar herramientas no registradas o no permitidas | P0 |
| FR-005 | Validar argumentos antes de cualquier ejecucion | P0 |
| FR-006 | Aplicar reglas de negocio fuera del texto del modelo | P0 |
| FR-007 | Clasificar herramientas por riesgo y requisitos de aprobacion | P0 |
| FR-008 | Vincular aprobacion a herramienta, argumentos, identidad, ejecucion y expiracion | P0 |
| FR-009 | Revalidar politica justo antes de ejecutar una accion aprobada | P0 |
| FR-010 | Evitar ejecuciones duplicadas tras reintentos o reanudaciones | P0 |
| FR-011 | Pausar y reanudar ejecuciones con estado durable | P0 |
| FR-012 | Emitir eventos de auditoria para decisiones y transiciones importantes | P0 |
| FR-013 | Redactar secretos y contenido sensible de logs y trazas | P0 |
| FR-014 | Declarar el soporte funcional de cada adaptador y proveedor | P1 |
| FR-015 | Exportar metricas de latencia, errores, tokens y coste | P1 |
| FR-016 | Ejecutar y comparar evals sobre datasets versionados | P1 |
| FR-017 | Inicializar proyectos desde plantillas versionadas | P1 |
| FR-018 | Cargar skills solo tras validar manifiesto, permisos y compatibilidad | P1 |
| FR-019 | Aplicar controles de retencion y borrado verificables | P1 |
| FR-020 | Crear una vista de operador para buscar estado y solicitudes pendientes | P2 |

P0 significa necesario para la primera plataforma que acepte acciones externas; P1 amplía la experiencia de desarrollo y evaluacion; P2 requiere validar necesidades reales antes de iniciar.

## 6. Requisitos no funcionales

### Seguridad

- Denegacion por defecto y separacion entre identidad, intencion del modelo y autorizacion.
- Proteccion contra replays, cambios de payload tras aprobacion y ejecucion concurrente.
- Secretos fuera de repositorio, logs y contexto enviado al modelo.
- Almacenamiento y telemetria con limites de datos documentados.
- Politicas y capacidades examinables antes del despliegue.

### Fiabilidad

- Reanudacion explicita, estados terminales y tratamiento de fallos ambiguos.
- Herramientas con timeout, limites de tamano y politica de reintentos por tipo de operacion.
- Una operacion con efecto lateral requiere soporte idempotente o un mecanismo equivalente de reconciliacion.

### Testabilidad

- Pruebas unitarias sin red, proveedor ni credenciales.
- Pruebas de contratos para herramientas, stores y proveedores.
- Pruebas integradas aisladas para persistencia, MCP y reanudacion.
- Evals con dataset, configuracion y version de modelo identificables.

### Mantenibilidad

- Dependencias por modulo y contratos documentados.
- Compatibilidad y politica de deprecacion declaradas.
- Logs y errores accionables sin tragar excepciones ni ofrecer fallos como respuestas exitosas.

### Portabilidad

- El entorno de desarrollo local debe funcionar con herramientas de prueba.
- Proveedores y adaptadores se incorporan bajo interfaces que expresan capacidades reales, no bajo una promesa de paridad total.

## 7. Alcance de la primera version de plataforma

### Incluido

- Un runtime Python modular.
- Dos casos de uso que reutilicen al menos una parte sustancial del runtime.
- Adaptador de modelo de referencia y fake determinista.
- Herramientas MCP con manifiesto local, permisos, validacion y politica de aprobacion.
- Un backend durable de estado mas un modo local sencillo.
- Evals reproducibles, trazas redactadas y CI.
- CLI de inicializacion/validacion cuando los contratos esten estables.

### Excluido

- Entrenamiento o hosting de modelos propios.
- Agentes que se coordinen libremente sin un objetivo y un limite definido.
- Ejecucion arbitraria de shell como herramienta por defecto.
- Un marketplace con contenido no verificado.
- Multi-tenancy comercial, facturacion o SLA antes de probar adopcion.
- Soporte de paridad para todos los IDE, agentes de codigo y proveedores.

## 8. Criterios de producto

La plataforma no se considera validada por el numero de paquetes, agentes o integraciones. Debe demostrar:

1. **Menor tiempo de arranque:** un desarrollador puede producir un nuevo ejemplo funcional con menos pasos que copiando la aplicacion base.
2. **Controles reusables:** el segundo agente no duplica el codigo de aprobacion, permisos, trazas y persistencia.
3. **Comportamiento seguro observable:** las pruebas comprueban que las acciones prohibidas no se ejecutan incluso si el modelo las solicita.
4. **Operabilidad:** una ejecucion puede explicarse, detenerse, reanudarse y reconciliarse.
5. **Calidad medida:** los cambios se comparan contra una linea base reproducible y se publican fallos conocidos.

## 9. Supuestos y preguntas que siguen abiertas

| Tema | Supuesto inicial | Decision que debe basarse en evidencia |
|---|---|---|
| Lenguaje del runtime | Python, por LangGraph/MCP y rapidez de iteracion | Si se mantiene como nucleo o se expone SDK Java |
| Orquestacion | LangGraph como primer adaptador | Si el contrato del runtime debe abstraer mas de un motor |
| Persistencia | SQLite/local para desarrollo; PostgreSQL candidato para despliegue | Almacen requerido tras probar reanudacion y concurrencia |
| Interfaz de aprobacion | CLI/API al comienzo | Si hace falta UI dedicada tras observar el flujo |
| Proveedores | Un proveedor real, mas fake local | Segundo proveedor cuando haya una necesidad funcional concreta |
| Integracion de IDE | Una integracion primero | Elegir segun feedback de usuarios objetivo |
| Datos de evaluacion | Casos sinteticos, no datos de empleador | Como obtener datasets realistas sin informacion sensible |

No cerrar estas decisiones por preferencia tecnologica: crear un ADR con contexto, alternativas, consecuencias y evidencia.
