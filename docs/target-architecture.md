# Arquitectura objetivo

> **Estado del documento:** propuesta. Describe el destino deseado, no componentes que ya esten implementados.

## 1. Principios de descomposicion

- El runtime coordina; el dominio define lo que significa una operacion valida.
- Los modelos producen propuestas estructuradas; el motor de politicas toma decisiones de autorizacion.
- El registro de herramientas es una fuente local de contratos y permisos; la metadata remota no puede ampliar autoridad.
- La ejecucion es una maquina de estados persistente, no una llamada HTTP sin historial.
- Interfaces estables separan los paquetes, pero no eliminan la necesidad de probar compatibilidad entre versiones.
- Persistencia, auditoria y telemetria son capacidades transversales con politicas de datos explicitadas.

## 2. Vista de componentes

```text
┌──────────────────── Superficies ────────────────────┐
│ CLI │ API de ejecucion │ Adaptadores de IDE futuros │
└──────────────────────────┬──────────────────────────┘
                           │ contrato de ejecucion
┌──────────────────────────▼──────────────────────────┐
│ Agent Runtime                                       │
│ ciclo de vida · limites · cancelacion · estado      │
└───────┬───────────────┬────────────────┬────────────┘
        │               │                │
┌───────▼───────┐ ┌─────▼────────┐ ┌─────▼────────────┐
│ Model Gateway │ │ Policy Engine│ │ Execution Store  │
│ proveedores  │ │ auth/approval│ │ checkpoints      │
└───────────────┘ └─────┬────────┘ └──────────────────┘
                        │ decision firmada/logica
┌───────────────────────▼─────────────────────────────┐
│ Tool & Skill Registry                               │
│ manifiestos · esquemas · versiones · capacidades    │
└───────────────────────┬─────────────────────────────┘
                        │
                MCP / conectores
                        │
                  sistemas externos

Auditoria, evaluacion y OpenTelemetry cubren cada frontera.
```

La expresion "decision firmada/logica" indica una decision vinculada a un payload inmutable y una identidad; no presupone una infraestructura de firma criptografica concreta.

## 3. Responsabilidad de cada modulo

### Agent Runtime

Posee el ciclo de vida de una ejecucion: aceptar solicitud, construir contexto, invocar el planificador, validar la propuesta, aplicar politica, ejecutar o pausar, guardar transicion y producir resultado. Aplica limites de iteraciones, coste, tiempo, tamano y cancelacion.

No posee reglas de dominio como "solo se reembolsa un pedido entregado"; esas reglas pertenecen al servicio de dominio.

### Model Gateway / Provider adapters

Normaliza operaciones necesarias del proveedor, declara capacidades (structured output, streaming, uso de herramientas) y convierte errores externos a errores internos tipados. Mantiene fuera del API comun opciones especificas que no puedan traducirse con fidelidad.

No autoriza herramientas, persiste conversaciones por defecto ni escribe contenido completo en telemetria.

### Policy Engine

Evalua identidad, agente, herramienta, accion, argumentos normalizados, contexto y politica versionada. Devuelve una decision explicita: `allow`, `deny` o `require_approval`, con identificador de politica y razon no sensible.

Para una accion sensible, la decision se repite justo antes de ejecutar; una aprobacion no reemplaza la comprobacion final.

### Tool Registry y Tool Executor

El registro valida manifiestos, versiones, esquemas, capacidades y clasificacion de riesgo. El ejecutor aplica serializacion, timeout, limites de payload, idempotency key, autenticacion del conector y normalizacion de resultados.

La metadata del servidor MCP puede ayudar al descubrimiento, pero nunca modifica en tiempo de ejecucion la lista local de permisos aprobados.

### Approval Service

Guarda solicitudes en estado `pending`, `approved`, `rejected`, `expired` o `consumed`. La aprobacion se liga al hash canonico de los parametros ejecutables, a la identidad del aprobador, al sujeto que inicio la ejecucion y a la expiracion. Cualquier alteracion invalida la decision.

### Execution Store

Persiste checkpoints y transiciones de estado con versionado optimista o bloqueo equivalente. Separa estado operacional de memoria semantica y define retencion y borrado por tipo de dato.

### Audit y observabilidad

Auditoria registra hechos de negocio y seguridad (quien, que decision, que referencia de ejecucion, cuando). OpenTelemetry registra tiempos, contadores y atributos tecnicos redactados. No son el mismo almacen ni deben tener politicas de retencion identicas.

### Evaluation Harness

Ejecuta casos fijos con modelo simulado y escenarios con proveedor real; compara trayectorias, decision de herramientas, cumplimiento de politica y resultado de negocio. Produce un artefacto con versiones y configuracion.

### CLI y adaptadores

La CLI inicializa, valida, ejecuta y evalua agentes. Adaptadores de IDE, API o sistemas de CI son capas externas: no contienen copias independientes del motor de permisos.

## 4. Modelo de ejecucion

Estados conceptuales:

```text
created -> planning -> validating -> policy_check
                                  -> denied -> completed
                                  -> approval_pending
approval_pending -> rejected -> completed
approval_pending -> expired -> completed
approval_pending -> approved -> policy_recheck -> executing
executing -> succeeded -> completed
executing -> failed | outcome_unknown -> reconciliation_required
```

`outcome_unknown` es necesario cuando el conector pudo ejecutar la accion pero no se recibio confirmacion. No debe reintentarse automaticamente salvo que el sistema destino soporte idempotencia o exista una reconciliacion comprobable.

Cada transicion debe incluir: identificador de ejecucion, version de agente, actor (cuando corresponda), marca temporal, estado previo y nuevo, referencia a politica, codigo de resultado y metadatos minimos.

## 5. Flujo de lectura

1. La superficie valida identidad, limites y configuracion del agente.
2. Runtime solicita al proveedor una propuesta dentro del contrato permitido.
3. Runtime valida nombre de herramienta y argumentos contra el manifiesto local.
4. Policy Engine determina si la lectura esta permitida para ese actor y contexto.
5. Tool Executor invoca MCP/conector con timeout y limites.
6. Resultado externo se marca como dato no confiable y se reduce al contexto necesario.
7. El modelo compone respuesta y el runtime aplica controles de salida definidos por el agente.
8. Se guarda resultado y auditoria; la telemetria conserva solo campos autorizados.

## 6. Flujo de escritura con aprobacion

1. El modelo propone una llamada, que no tiene capacidad de ejecutar directamente.
2. El runtime verifica el esquema, normaliza valores y calcula la accion que se presentara.
3. Policy Engine consulta las reglas deterministas y exige aprobacion.
4. Approval Service crea solicitud con payload inmutable, hash canonico, identidad, expiracion y clave de ejecucion.
5. Runtime persiste estado y pausa; la API devuelve un identificador de solicitud, no una falsa confirmacion de exito.
6. Aprobador ve herramienta, efecto, argumentos relevantes, consecuencias y tiempo restante.
7. Al aceptar, runtime verifica identidad, expiracion, hash, version de politica y estado de ejecucion.
8. Se evalua de nuevo la politica; si algo cambio, se rechaza o vuelve a aprobacion con una nueva solicitud.
9. Tool Executor ejecuta usando clave idempotente cuando el destino lo soporte.
10. Se persiste resultado. Si la respuesta del destino es ambigua, el estado requiere reconciliacion; no se afirma que la operacion fallo o tuvo exito sin evidencia.

## 7. Contratos internos a definir

Las siguientes firmas son conceptos de dominio, no API publica final:

```text
AgentDefinition:
  id, version, prompt_ref, allowed_tool_refs, policy_ref,
  model_profile_ref, limits, data_classification

ToolManifest:
  id, version, input_schema, output_schema, capabilities,
  risk_class, required_permissions, timeout, idempotency_mode

PolicyDecision:
  outcome, policy_id, policy_version, reason_code,
  approval_requirements, expires_at

Execution:
  execution_id, tenant_or_subject, agent_ref, state,
  created_at, updated_at, checkpoint_ref, result_ref

ApprovalRequest:
  approval_id, execution_id, action_digest, requested_by,
  approver_scope, expires_at, status, decision_ref
```

Definir tipos y validaciones antes de elegir nombres de endpoints. Los contratos publicos deben pasar por compatibilidad y versionado.

## 8. Datos y clasificacion

| Dato | Uso | Tratamiento inicial |
|---|---|---|
| Configuracion publica del agente | Describir flujo y limites | Versionada; secretos solo por referencia |
| Credencial del proveedor/conector | Autenticar llamada | Secret manager o variable local ignorada por Git; nunca prompt/log |
| Entrada de usuario | Solicitud de tarea | Minimizar, limitar y clasificar antes de persistir |
| Resultado de herramienta | Evidencia para resolver tarea | Dato no confiable; filtrar y limitar contexto |
| Checkpoint | Reanudar ejecucion | Retencion definida; cifrado segun entorno |
| Aprobacion | Autorizar accion especifica | Inmutable respecto al payload aprobado; vencimiento |
| Auditoria | Investigar decision y ejecucion | Referencias y codigos; evitar payload completo |
| Telemetria | Medir salud y coste | Atributos tecnicos redactados; muestreo y retencion |
| Evals | Medir regresion | Datos sinteticos o anonimizados con origen documentado |

La clasificacion, jurisdiccion, cifrado en reposo y retencion exacta deben cerrarse antes de almacenar datos reales de clientes.

## 9. Decisiones tecnicas iniciales y pruebas de salida

| Tema | Propuesta inicial | Validacion antes de comprometerse |
|---|---|---|
| Lenguaje | Python en runtime; Java puede integrar por API despues | Segundo caso, tipos, empaquetado y skill set del usuario |
| Orquestacion | Mantener LangGraph como primer motor | Que sus checkpoints y transiciones soporten los requisitos definidos |
| Protocolo de herramientas | MCP para conectores interoperables | Identidad, permisos, timeout y errores por transporte |
| Persistencia | Interfaz propia; adaptador local para desarrollo | Pruebas de reinicio, concurrencia, migracion y borrado |
| Evaluacion | pytest mas harness de trayectorias versionado | Reproducibilidad y costo operativo de pruebas con LLM |
| Observabilidad | OpenTelemetry | Semantica estable y redaccion efectiva en todos los exporters |
| CLI | Herramienta Python empaquetada | Experiencia limpia en Windows, Linux y CI |
| UI | Diferida; API/CLI primero | Friccion real de operadores para aprobar y recuperar |

Registrar las decisiones finales como ADRs con contexto, decision, alternativas, consecuencias y pruebas.

## 10. Fronteras de confianza

1. **Cliente a superficie:** identidad, autenticacion, cuota y validacion de entrada.
2. **Runtime a modelo:** el prompt y la respuesta son entrada/salida no confiable.
3. **Runtime a servidor MCP:** transporte remoto, herramienta, esquema y resultado requieren validacion.
4. **Aprobador a Approval Service:** identidad y rol del aprobador se verifican independientemente de quien solicito la accion.
5. **Runtime a store:** acceso aislado por ejecucion y sujeto, control de concurrencia y retencion.
6. **Aplicacion a telemetria:** filtrado antes de exportar; no depender solo de privacidad del backend de observabilidad.

## 11. Evolucion sin big bang

- Mantener primero el agente actual ejecutable.
- Extraer un contrato a la vez y proveer adaptadores para la implementacion existente.
- Crear un segundo agente consumidor antes de declarar un paquete reusable.
- Comparar las trayectorias y resultados del agente antes y despues de cada extraccion.
- Versionar API y datos; migraciones deben ser reversibles o tener plan probado de recuperacion.
- Evitar que el framework conozca nombres de dominio como pedido, reembolso o cliente.
