# Guía conceptual: sistemas de agentes con controles verificables

## Propósito

Esta guía presenta los conceptos necesarios para entender un agente que interactúa con sistemas y puede proponer acciones con efectos. Usa el agente de operaciones de pedidos como ejemplo práctico y la plataforma de agentes como contexto de evolución. No presupone que una arquitectura propuesta ya esté implementada: el código y las pruebas de cada repositorio son la fuente de evidencia de su estado actual.

La [ruta de aprendizaje](learning-path.md) organiza una secuencia de estudio entre los materiales y repositorios afines. Esta guía se centra en el vocabulario y los modelos mentales que ayudan a recorrerla.

## La idea central

Un modelo genera una propuesta. El sistema decide qué contexto darle, qué capacidades exponer, cómo validar la propuesta y si la acción puede ejecutarse. Para operaciones sensibles, debe haber reglas deterministas, autorización y evidencia fuera del modelo.

```text
solicitud
  -> interpretar y proponer
  -> validar herramienta y argumentos
  -> comprobar política y reglas de negocio
  -> pedir aprobación si corresponde
  -> ejecutar con límites
  -> registrar resultado y verificarlo
```

Por eso, construir agentes no es sólo redactar mejores prompts: también es diseñar contratos, permisos, control de flujo, manejo de fallos, pruebas y operación.

## 1. Vocabulario esencial

| Término | Significado práctico | No significa |
|---|---|---|
| **Modelo** | Componente que produce texto o una propuesta estructurada a partir de una entrada. | Que tenga permisos propios o que pueda demostrar que su respuesta es correcta. |
| **Agente** | Aplicación que combina un modelo con un ciclo de control, contexto y capacidades para avanzar hacia un objetivo. | Un sistema autónomo, infalible o responsable de sus efectos. |
| **Runtime** | Código que coordina la ejecución, el estado, los límites y la interacción con modelos y herramientas. | Una política segura por el solo hecho de existir. |
| **Planner** | Componente que propone la respuesta o el siguiente paso. Puede delegar esa propuesta a un modelo. | La autoridad para ejecutar la acción propuesta. |
| **Herramienta** | Operación concreta, como consultar un pedido o solicitar un reembolso. | Un permiso. Que una operación exista no implica que cualquier agente pueda usarla. |
| **Manifiesto** | Declaración de herramientas, esquemas y capacidades disponibles para un agente. | Validación suficiente de identidad, argumentos o reglas de negocio. |
| **Política** | Decisión determinista sobre permitir, denegar o exigir aprobación. | Una frase del prompt ni la intención inferida del modelo. |
| **MCP** | Protocolo para conectar aplicaciones con herramientas o fuentes de contexto. | Una garantía de confianza, autorización o seguridad del servidor conectado. |
| **Aprobación humana** | Decisión de una persona autorizada sobre una acción concreta y sus argumentos. | Un permiso permanente para que el agente realice acciones similares en el futuro. |
| **Evaluación de trayectoria** | Prueba del comportamiento del agente a través de una secuencia de decisiones. | Una garantía de que todos los modelos o ejecuciones futuras se comportarán igual. |

Los nombres y capacidades exactos dependen de la tecnología. Antes de aplicar un concepto a un framework, verifica sus contratos y documentación vigentes.

## 2. Modelo mental: propuesta, decisión y efecto

Al analizar cualquier agente, separa tres momentos:

1. **Propuesta:** el modelo sugiere una respuesta o una llamada a herramienta.
2. **Decisión:** el runtime y los servicios deterministas validan si esa operación está permitida, con esos argumentos y en este contexto.
3. **Efecto:** un ejecutor llama a la herramienta o sistema externo y obtiene un resultado que debe manejarse.

No se deben confundir. Una llamada sugerida no es una llamada autorizada; una llamada autorizada tampoco demuestra que el efecto externo haya tenido éxito. Los errores y los resultados ambiguos necesitan tratamiento explícito.

### Ejemplo breve

Una persona pide revisar un pedido y devolver 20 unidades monetarias. El modelo puede proponer `request_refund(order_id, amount)`, pero el sistema debe comprobar, como mínimo:

- que la herramienta existe y está en la lista permitida para este agente;
- que el identificador y el importe cumplen el esquema esperado;
- que la identidad y las reglas de negocio permiten la operación;
- que se obtuvo la aprobación requerida para **esos argumentos exactos**;
- que el ejecutor devuelve un resultado verificable.

La decisión de negocio no se delega al modelo. El prompt puede explicar las reglas, pero el código que autoriza y ejecuta debe volver a comprobarlas.

## 3. Componentes y responsabilidades

Una arquitectura de referencia puede contener:

| Componente | Responsabilidad principal | Pregunta de revisión |
|---|---|---|
| Interfaz o API | Recibir una solicitud e identidad. | ¿Quién hace la solicitud y con qué permisos? |
| Orquestación o grafo | Controlar transiciones, límites y pausas del flujo. | ¿Qué estados y transiciones son válidos? |
| Planner/modelo | Proponer una respuesta o herramienta. | ¿Qué salida estructurada se acepta y qué pasa si es inválida? |
| Manifiesto/registro | Describir las capacidades disponibles. | ¿La lista efectiva se valida en ejecución? |
| Política y dominio | Autorizar acciones y aplicar reglas del negocio. | ¿Las reglas se comprueban fuera del modelo? |
| Aprobador | Revisar una acción sensible antes de ejecutarla. | ¿Ve exactamente los argumentos que se ejecutarán? |
| Ejecutor | Invocar herramienta local, API o servidor MCP. | ¿Hay límites, timeout, manejo de errores e idempotencia? |
| Estado y auditoría | Conservar el progreso y los hechos necesarios para investigar. | ¿Se puede reanudar sin duplicar efectos ni guardar datos excesivos? |
| Pruebas y evaluación | Comprobar invariantes y observar el comportamiento. | ¿Qué afirmación demuestra cada prueba y qué no demuestra? |

Los componentes pueden estar reunidos en menos módulos. Lo importante es que las responsabilidades sean identificables y se puedan probar.

## 4. Aprobaciones vinculadas a la acción

Una aprobación útil no dice «confío en este agente». Presenta una acción concreta —operación, destino y argumentos relevantes— para que una persona autorizada la acepte o rechace.

El sistema debería verificar que:

- la solicitud de aprobación describe la misma acción que se ejecutará;
- cualquier modificación de argumentos invalida la aprobación anterior;
- se valida la identidad y autoridad de quien aprueba;
- la aprobación tiene un alcance y una vigencia definidos;
- rechazo, expiración y error no se convierten en aprobación;
- reanudar el flujo no repite un efecto ya realizado.

La experiencia de aprobación debe mostrar información suficiente para decidir, pero no revelar secretos o datos personales innecesarios.

## 5. MCP y límites de confianza

MCP permite separar al consumidor de una herramienta de la implementación del servidor. Esa separación facilita la integración, pero agrega una frontera que hay que validar.

- Un servidor puede devolver datos o descripciones incorrectos o maliciosos.
- El nombre de una herramienta remota no concede permiso para ejecutarla.
- El manifiesto y la política locales deben controlar qué herramientas puede invocar el agente.
- Los argumentos se validan en el runtime antes de llegar al ejecutor.
- Los errores del servidor deben tratarse como entradas externas, no como instrucciones.
- La identidad, el transporte y los permisos del servidor requieren configuración y controles propios.

Una defensa no se basa en pedirle al modelo que ignore instrucciones hostiles; limita técnicamente qué efectos pueden producirse aunque el modelo las siga.

## 6. Fallos que el diseño debe contemplar

| Situación | Riesgo | Respuesta de diseño |
|---|---|---|
| Herramienta desconocida | Invocación fuera del alcance previsto. | Denegar antes de llamar al ejecutor. |
| Argumento inválido | Acción sobre un objeto o importe incorrecto. | Validar esquema y reglas de dominio antes de ejecutar. |
| Aprobación para argumentos distintos | Efecto no revisado por la persona. | Vincular aprobación al payload y rechazar cambios posteriores. |
| Timeout del sistema externo | No se sabe si la acción se aplicó. | Registrar estado ambiguo y reconciliar; no reintentar a ciegas. |
| Reinicio durante una aprobación | Pérdida de contexto o repetición de operación. | Persistir estado de forma segura y definir expiración y reanudación. |
| Reintento concurrente | Efecto duplicado. | Usar idempotencia y protección del lado del destino cuando sea posible. |
| Instrucción hostil en datos | El modelo puede tratarla como orden. | Tratar datos externos como no confiables y contener efectos con controles deterministas. |
| Bucle de planificación | Coste y ejecución sin progreso. | Límites de pasos, tiempo y coste con resultado explícito al agotarse. |

Persistir un checkpoint no resuelve automáticamente la consistencia con un sistema externo. Si una petición externa pudo completarse antes de un fallo, el runtime necesita una estrategia de reconciliación.

## 7. Pruebas y evaluaciones: qué evidencia aporta cada una

| Evidencia | Ayuda a comprobar | No demuestra por sí sola |
|---|---|---|
| Unit test de dominio | Reglas de negocio para entradas concretas. | Seguridad completa del flujo ni del proveedor. |
| Test del runtime | Transiciones, límites y denegaciones implementadas. | Comportamiento de un servidor externo real. |
| Test de contrato MCP | Compatibilidad del contrato probado. | Confianza general en todos los servidores MCP. |
| Test de integración | Interacción entre componentes concretos. | Comportamiento de todos los modelos y configuraciones. |
| Evaluación de trayectoria | Decisiones observadas en escenarios identificados. | Invariantes deterministas para cualquier entrada futura. |
| Revisión y auditoría | Contexto, decisiones y evidencia disponibles. | Ausencia de errores no observados. |

Las reglas críticas —autorización, límites financieros, herramientas permitidas— deben tener comprobaciones deterministas. Las evaluaciones con modelos complementan estas pruebas; no las sustituyen. Reporta modelo, versión, configuración, conjunto de casos y resultados cuando publiques métricas.

## 8. Auditoría, telemetría y privacidad

Auditoría y telemetría responden preguntas distintas:

- **Auditoría:** quién pidió, aprobó o ejecutó una acción y cuál fue el resultado.
- **Telemetría:** cuánto tardó, dónde falló y qué recursos consumió el sistema.

No es necesario guardar prompts y respuestas completos para instrumentar un flujo. Prefiere identificadores de ejecución, nombres de operación, estados, duración y errores normalizados. Define acceso, retención y redacción; no registres credenciales ni datos de pago en trazas.

## 9. Cómo razonar sobre una plataforma reusable

Un agente concreto responde a un caso. Una plataforma añade contratos y componentes compartidos para construir más de un agente. Esa generalización sólo está justificada cuando un segundo caso real confirma que la abstracción funciona fuera del primero.

Al evaluar una extracción, pregunta:

1. ¿Qué responsabilidad se repite realmente?
2. ¿Qué contrato necesita el consumidor, incluidos errores y límites?
3. ¿Puede un fake determinista probarlo sin credenciales ni red?
4. ¿Qué capacidades difieren entre proveedores o ejecutores?
5. ¿Existe un segundo consumidor que demuestre la reutilización?
6. ¿La abstracción hace más fácil mantener controles, o sólo añade capas?

Un diseño objetivo, un hito de roadmap o una interfaz aislada son propuestas hasta que la evidencia correspondiente confirma su comportamiento. Consulta el [roadmap de plataforma](https://github.com/mauricio-acuna/agent-platform-roadmap/blob/main/ROADMAP.md) y el [código del agente de operaciones](https://github.com/mauricio-acuna/ops-agent-mcp) para distinguir estados.

## 10. Cómo estudiar el ejemplo de operaciones

En el repositorio de referencia, sigue una solicitud de principio a fin:

1. Lee el [README del agente](https://github.com/mauricio-acuna/ops-agent-mcp/blob/main/README.md) para entender alcance y límites.
2. Recorre su [guía de aprendizaje](https://github.com/mauricio-acuna/ops-agent-mcp/blob/main/docs/learning-guide.md) para llegar al código, las pruebas y las limitaciones.
3. Traza cómo el planner produce una propuesta, cómo el flujo la valida y cuándo se requiere aprobación.
4. Comprueba qué reglas residen en el dominio y cuáles en la política.
5. Identifica qué pruebas ejecutan sin proveedor externo y cuáles requieren configuración adicional.
6. Distingue la telemetría de las decisiones auditadas.

La aplicación de referencia usa pedidos y reembolsos ficticios como material de aprendizaje; no debe confundirse con un sistema de pagos productivo.

## Glosario breve

- **Allowlist:** lista explícita de capacidades permitidas.
- **Checkpoint:** estado conservado para continuar un flujo; no garantiza consistencia por sí solo.
- **Idempotencia:** propiedad que evita duplicar un efecto al repetir una operación compatible con esa protección.
- **Resultado ambiguo:** situación en la que el sistema no sabe si el servicio externo completó la acción.
- **Runtime:** entorno ejecutable que coordina el ciclo y aplica límites.
- **Tool call:** propuesta estructurada para invocar una herramienta; no es una autorización.

## Fuentes y lecturas relacionadas

- [Ops Agent + MCP](https://github.com/mauricio-acuna/ops-agent-mcp) — implementación de referencia y evidencia de pruebas.
- [Guía de aprendizaje del agente](https://github.com/mauricio-acuna/ops-agent-mcp/blob/main/docs/learning-guide.md) — recorrido detallado por el código, las prácticas y las limitaciones.
- [Roadmap de plataforma](https://github.com/mauricio-acuna/agent-platform-roadmap) — visión del producto, arquitectura propuesta y estado de los hitos.
- [Ruta de aprendizaje](learning-path.md) — unidades, actividades y proyecto final.
- [Calidad y seguridad](quality-and-security.md) — amenazas, controles, estrategia de pruebas y gates de release.
- [Definición de producto](product-definition.md) — usuarios, casos de uso y límites del MVP.
