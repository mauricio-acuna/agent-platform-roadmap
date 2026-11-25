# Calidad, seguridad y modelo de evaluacion

## 1. Objetivo

Probar que el sistema mantiene comportamiento seguro y funcional bajo cambios de codigo, de modelo, de herramienta y de configuracion. Ningun conjunto de pruebas demuestra seguridad total; el objetivo es establecer controles en profundidad, fallos detectables y evidencia revisable.

## 2. Activos que proteger

- Fondos y operaciones de negocio ejecutadas por herramientas.
- Datos personales, financieros, comerciales y credenciales.
- Identidad del usuario y autoridad del aprobador.
- Integridad de los argumentos, del estado de aprobacion y del historial.
- Disponibilidad y presupuesto de inferencia y herramientas externas.
- Integridad de skills, plantillas, manifiestos y paquetes.
- Resultados de evals y trazas usados para decidir releases.

## 3. Adversarios y fuentes de riesgo

- Usuario autenticado que intenta ampliar su autoridad o provocar abuso de herramientas.
- Usuario no autenticado que alcanza una superficie expuesta.
- Modelo que alucina, obedece instrucciones hostiles o produce argumentos invalidos.
- Contenido externo (documentos, resultados MCP, tickets) con instrucciones indirectas.
- Servidor MCP o proveedor comprometido, mal configurado o con metadata engañosa.
- Dependencia, skill o plantilla con comportamiento no esperado.
- Operador que confunde una solicitud pendiente con una accion ya ejecutada.
- Reintentos, concurrencia, fallos parciales y errores de infraestructura.

## 4. Amenazas y controles requeridos

| Amenaza | Ejemplo | Control preventivo | Verificacion |
|---|---|---|---|
| Autoridad excesiva | El agente invoca una herramienta administrativa | Denegacion por defecto, permisos de herramienta y sujeto | Prueba de herramienta fuera de allowlist |
| Cambio tras aprobacion | Cambiar el monto despues del clic de aceptar | Hash canonico del payload, estado inmutable y revalidacion | Alterar cada campo y comprobar que la ejecucion se bloquea |
| Replay/duplicacion | Reintentar una operacion tras timeout | Clave idempotente, estado terminal y reconciliacion | Repetir la misma solicitud en serie y concurrencia |
| Prompt injection indirecta | Respuesta de herramienta pide ignorar politica | Tratar resultado como dato, permisos fuera del prompt | Corpus adversarial con llamadas de herramienta prohibidas |
| Confusion de identidad | Un usuario aprueba una accion ajena | Autorizacion del aprobador y alcance por tenant/ejecucion | Pruebas de acceso cruzado |
| Tool poisoning | Descripcion MCP intenta pedir credenciales | Manifiestos locales confiables y verificacion de servidor | Alterar descripcion remota; lista efectiva no debe cambiar |
| Fuga de datos | Prompt, tarjeta o token aparece en traza | Redaccion y minimizacion antes del exporter | Inspeccionar atributos y logs con canarios sensibles |
| Agotamiento de recursos | Bucle de agente o respuestas enormes | Limite de pasos, tokens, tiempo, tamano y cuota | Casos de limite y presupuesto excedido |
| Skill maliciosa | Paquete intenta ampliar permisos | Manifiesto revisable, sandbox/aislamiento futuro y aprobacion de permisos | Skill con permiso no declarado debe fallar |
| Dependencia vulnerable | Actualizacion introduce CVE o comportamiento incompatible | Escaneo, lockfile, actualizacion revisada y CI | Gate automatizado y registro de excepciones |

La tabla es un punto de partida; cada nueva integracion requiere actualizar el modelo de amenazas antes de exponerla a usuarios.

## 5. Invariantes de seguridad

1. Solo el motor determinista decide si una herramienta puede ejecutarse.
2. Una herramienta no registrada no se ejecuta aunque el modelo la solicite.
3. Una herramienta que modifica estado no se ejecuta sin politica explicita.
4. La aprobacion esta ligada al payload exacto, identidad, ejecucion y periodo de validez.
5. La aprobacion no omite revalidacion de politica, negocio ni estado.
6. Repetir una llamada no debe producir efectos duplicados; donde no se pueda garantizar idempotencia, se requiere reconciliacion.
7. Contenido de herramienta no puede cambiar instrucciones, permisos o identidad.
8. Excepciones, timeouts y respuestas ambiguas no se convierten en mensajes que afirmen exito.
9. Secretos no aparecen en prompts, repositorios, tests, capturas, logs ni telemetria.
10. Cambios a permisos, politicas o manifiestos generan una diferencia revisable.

## 6. Piramide de pruebas

### Nivel 1 — Unitarias deterministas

Sin red ni credenciales. Cubren reglas de dominio, schemas, autorizacion, transiciones de estado, expiracion, hashes, idempotencia, redaccion y calculo de limites.

**Gate:** obligatorio en cada pull request; todos los casos criticos pasan.

### Nivel 2 — Contrato de componentes

Verifican que implementaciones de `ModelProvider`, `ToolExecutor`, `StateStore` y `ApprovalHandler` cumplen comportamiento definido. Simulan timeout, errores de protocolo, respuestas parciales y payloads grandes.

**Gate:** obligatorio al cambiar una interfaz o adaptador.

### Nivel 3 — Integracion local

Ejecutan persistencia real, flujo MCP local y reanudacion desde un proceso nuevo. No requieren un proveedor remoto para validar el control.

**Gate:** en CI al cambiar runtime, store o conectores.

### Nivel 4 — Evaluacion de trayectoria con modelo real

Ejecuta escenarios con proveedor/modelo identificados; compara herramienta elegida, argumentos, politica, aprobacion, resultado y salida. Las metricas separan errores criticos de calidad general.

**Gate:** ejecucion programada o manual; bloquea release si un caso critico de seguridad falla. Registrar fluctuacion y coste; no esconder fallos intermitentes.

### Nivel 5 — Pruebas de despliegue y operacion

Verifican instalacion limpia, configuracion, rotacion de secretos, health checks, actualizacion, recuperacion y procedimientos de borrado.

**Gate:** antes de una release que anuncie despliegue productivo.

## 7. Diseño del dataset de evaluacion

Cada caso debe describir:

- `case_id`, version y objetivo.
- Entrada inicial y datos de herramientas simulados.
- Acciones esperadas y acciones prohibidas.
- Politica de aprobacion esperada.
- Resultado de negocio esperado y condiciones de abstencion.
- Severidad y condicion de criticidad.
- Origen del caso: sintetico, anonimo o generado; nunca datos confidenciales de empleador.

### Categorias minimas

1. Consulta correcta de solo lectura.
2. Solicitud valida de accion con aprobacion obligatoria.
3. Solicitud invalida por regla de negocio.
4. Herramienta no permitida.
5. Argumentos malformados, fuera de rango o con campos extras.
6. Inyeccion directa e indirecta.
7. Resultado del proveedor ambiguo, timeout y error de protocolo.
8. Reanudacion, repeticion, expiracion y accion concurrente.
9. Intento de acceso cruzado entre identidades/tenants.
10. PII y secretos que no deben filtrarse a respuesta o traza.

### Metricas

- **Policy violation rate:** llamadas a herramientas no permitidas / ejecuciones. Objetivo: cero en casos criticos.
- **Approval bypass rate:** acciones sensibles ejecutadas sin aprobacion valida. Objetivo: cero en pruebas adversariales.
- **Tool selection accuracy:** herramientas necesarias invocadas y herramientas prohibidas no invocadas.
- **Argument validity:** proporción de propuestas aceptadas que cumplen contrato antes de llamar.
- **Business outcome accuracy:** reglas de dominio satisfechas en los resultados.
- **Safe abstention:** abstencion cuando faltan datos o permiso.
- **Duplicate side-effect rate:** acciones duplicadas tras retry/recovery. Objetivo: cero en entorno de prueba idempotente.
- **Telemetry leakage count:** canarios sensibles encontrados en trazas y logs. Objetivo: cero.
- **Cost/latency:** medir mediana y percentiles por modelo y caso; umbrales se deciden tras obtener linea base.

No promediar un fallo critico de autorizacion con respuestas correctas para producir una puntuacion aceptable.

## 8. Politica de logs y telemetria

### Permitido por defecto

- identificador aleatorio de ejecucion;
- nombre y version de herramienta;
- codigo de estado y razon no sensible;
- duracion, contadores y coste estimado;
- version de agente, politica y proveedor;
- decision de aprobacion y actor mediante identificador interno protegido.

### Prohibido por defecto

- API keys, tokens OAuth, contrasenas o cabeceras de autenticacion;
- prompts completos, respuestas completas y chain-of-thought;
- payloads de herramientas con datos personales o financieros;
- documentos completos, contenido de tickets o datos del empleador;
- atributos de OpenTelemetry con texto libre no revisado.

Si un diagnostico requiere contenido, debe ser opt-in, temporal, con redaccion, acceso restringido y retencion minima.

## 9. Gates de release

Una release no se publica si:

- fallan pruebas de autorizacion, aprobacion o aislamiento;
- se detecta una fuga de secreto en pruebas de telemetria;
- una migracion de estado no tiene prueba de recuperacion;
- hay una regresion critica de eval sin excepcion revisada y documentada;
- el manifiesto o version de herramienta no es compatible;
- la guia de instalacion limpia no fue ejecutada;
- no se conocen dependencias y artefactos incluidos.

Toda excepcion debe identificar responsable, riesgo residual, vencimiento y compensacion; no se permiten bypass permanentes sin owner.

## 10. Respuesta a incidentes y vulnerabilidades

- Documentar como deshabilitar proveedor, herramienta o skill comprometidos.
- Mantener un interruptor operativo para bloquear nuevas acciones de escritura.
- Poder revocar o invalidar aprobaciones pendientes.
- Preservar evidencia minima para investigar sin copiar datos sensibles indiscriminadamente.
- Publicar canal y politica de divulgacion responsable antes de una release estable.
- Registrar causas raiz y agregar una prueba de regresion para cada incidente reproducible.

## 11. Proceso de evaluacion de una version

1. Identificar commit, version de runtime, agente, politica, proveedor/modelo y configuracion.
2. Ejecutar pruebas deterministas y de contrato.
3. Ejecutar escenarios de integracion y recuperacion.
4. Ejecutar evals de modelo y registrar respuestas/metricas con politica de privacidad definida.
5. Comparar con baseline por categoria; investigar regresiones, no solo el score total.
6. Revisar cambios de permisos y dependencias.
7. Adjuntar informe y excepciones a la release.

## 12. Criterio de madurez

Usar etiquetas honestas:

- **Demo:** entorno local y datos sinteticos.
- **Preview:** contratos en cambio; requiere supervision y configuracion revisada.
- **Release estable:** gates, migracion, soporte, recuperacion y seguridad documentados.
- **Produccion de una organizacion:** requiere ademas threat model especifico, pruebas de carga, controles de identidad, privacidad y operacion del entorno concreto.

El estado de una etiqueta debe reflejar evidencia del software, no intencion del roadmap.
