# Roadmap de implementacion

## Estado y convenciones

- **Pendiente:** no iniciado.
- **En curso:** trabajo activo con alcance y criterios definidos.
- **Bloqueado:** requiere una decision o resultado anterior.
- **Hecho:** criterios de aceptacion cumplidos y evidencia enlazada.

Todas las etapas comienzan como **Pendiente**. Cambiar el estado requiere evidencia en el repositorio de implementacion, una release, una prueba reproducible o un informe de evaluacion.

## Arquitectura objetivo

```text
CLI / adaptadores de IDE / API
              |
       Agent Runtime
       /     |      \
  modelos  politicas  memoria/estado
       \     |      /
       Tool & Skill Registry
              |
      MCP y conectores
              |
       sistemas externos

Evals + auditoria + OpenTelemetry atraviesan el runtime y los conectores.
```

Los limites entre componentes deben ser contratos pequenos y testeables. El runtime no debe asumir que el modelo, la herramienta o el adaptador son confiables.

## Hito 0 — Consolidar el agente base

**Estado:** Pendiente
**Dependencia:** ninguna
**Proposito:** convertir el agente existente en una demo verificable y una referencia de comportamiento para las siguientes etapas.

### Trabajo

- Ejecutar la suite completa en CI y conservar el resultado de referencia.
- Ejecutar las evaluaciones de trayectoria con modelos reales; documentar modelo, fecha, configuracion, pass rate y fallos criticos.
- Separar evaluaciones deterministas de pruebas que consumen API o requieren credenciales.
- Implementar persistencia durable del checkpoint y reanudacion de una aprobacion tras reiniciar el proceso.
- Preparar una demo corta que muestre consulta, aprobacion, rechazo y una accion bloqueada por regla de negocio.
- Documentar limites conocidos, incluyendo latencia, coste y condiciones de fallo.

### Criterios de aceptacion

- CI ejecuta pruebas sin credenciales externas.
- Evals reales producen un informe reproducible y nunca registran secretos ni contenido sensible.
- Una aprobacion pendiente puede recuperarse despues de reiniciar el servicio.
- Aprobar, rechazar, expirar o corromper un estado no puede ejecutar una accion dos veces.
- La demo se puede reproducir siguiendo unicamente el README.

### Entregable

Una version etiquetada del agente de operaciones con pruebas, demo y resultados de evaluacion publicados.

## Hito 1 — Extraer un runtime reutilizable

**Estado:** Pendiente
**Dependencia:** Hito 0

### Trabajo

- Definir contratos tipados para `Agent`, `Planner/ModelProvider`, `ToolExecutor`, `ApprovalHandler` y `StateStore`.
- Separar el flujo de negocio de pedidos del runtime de agente.
- Permitir configurar modelos y parametros mediante configuracion validada, sin acoplar el dominio a un proveedor.
- Mantener adaptadores simples; evitar una capa de abstraccion generica sin un segundo caso de uso real.
- Crear un segundo agente minimo que reuse el runtime y pruebe que la separacion es genuina.

### Criterios de aceptacion

- El agente de pedidos y el segundo ejemplo usan los mismos contratos del runtime.
- Pruebas unitarias pueden sustituir el modelo por un fake determinista.
- Ninguna regla de autorizacion depende de la respuesta del modelo.
- Los contratos y errores de configuracion estan documentados.

### Entregable

Paquete de runtime versionado, con ejemplo de referencia y guia para crear un agente nuevo.

## Hito 2 — Registro de herramientas y motor de politicas

**Estado:** Pendiente
**Dependencia:** Hito 1

### Trabajo

- Crear un registro de herramientas con identificador, version, esquema de entrada, permisos y clasificacion de riesgo.
- Validar argumentos y salida de herramientas en cada ejecucion, no solo al planear.
- Aplicar denegacion por defecto, allowlists y aprobacion obligatoria segun la politica.
- Verificar identidad y capacidades del servidor MCP conectado; no confiar en descripciones remotas para decidir permisos.
- Normalizar resultados y errores sin incluir secretos ni datos innecesarios en trazas.
- Especificar timeouts, reintentos, idempotencia y limites de tamano para llamadas externas.

### Criterios de aceptacion

- Una herramienta no registrada o sin permiso se bloquea antes de ejecutarse.
- Cambiar argumentos despues de una aprobacion invalida esa aprobacion.
- Las herramientas con efectos laterales requieren politica explicita y aprobacion.
- Contratos incompatibles fallan con un error claro y auditable.
- Pruebas cubren entradas malformadas, denegacion, timeout y repeticion segura.

### Entregable

Registro de herramientas MCP reutilizable y motor de politicas con ejemplos de solo lectura y de escritura.

## Hito 3 — Persistencia, reanudacion y memoria

**Estado:** Pendiente
**Dependencia:** Hitos 1 y 2

### Trabajo

- Definir un `StateStore` con una implementacion local y otra apta para despliegue compartido.
- Guardar checkpoints, aprobaciones y resultados con identificadores de ejecucion y control de concurrencia.
- Definir expiracion, borrado, aislamiento por usuario/tenant y politica de retencion.
- Distinguir memoria operacional (estado de una ejecucion) de memoria de conocimiento (datos consultables).
- No persistir prompts, secretos o informacion personal salvo necesidad documentada y controles adecuados.

### Criterios de aceptacion

- Reinicios y ejecuciones concurrentes no duplican acciones.
- Una ejecucion no puede leer o reanudar estado de otro tenant.
- Expiracion y borrado son verificables mediante pruebas.
- El modo local sigue disponible para demos sin infraestructura externa.

### Entregable

Almacenamiento durable intercambiable, especificacion de retencion y guia de despliegue.

## Hito 4 — Evaluacion, seguridad y observabilidad

**Estado:** Pendiente
**Dependencia:** Hitos 1–3

### Trabajo

- Mantener un conjunto versionado de tareas representativas y trayectorias esperadas.
- Medir por separado seleccion de herramienta, validez de argumentos, resultado de negocio, necesidad de aprobacion y calidad de respuesta.
- Agregar pruebas adversariales para prompt injection, exceso de autoridad, filtracion de datos, abuso de herramientas y errores de proveedor.
- Incorporar OpenTelemetry para latencia, tokens, errores y llamadas de herramientas, con redaccion de datos sensibles.
- Establecer gates de CI: regresiones deterministas bloquean; evals con LLM se etiquetan y comparan con una linea base.
- Publicar un informe que incluya limitaciones, modelo, coste aproximado y variabilidad.

### Criterios de aceptacion

- Cada cambio de runtime ejecuta los casos deterministas en CI.
- Ninguna traza contiene prompts completos, respuestas, tokens secretos o argumentos sensibles.
- Los casos criticos tienen una politica de bloqueo clara, no solo una puntuacion agregada.
- Un informe puede regenerarse desde el commit y configuracion identificados.

### Entregable

Harness de evaluacion y observabilidad reutilizable para los agentes del proyecto.

## Hito 5 — Skills, plantillas y paquetes

**Estado:** Pendiente
**Dependencia:** Hitos 1, 2 y 4

### Trabajo

- Definir formato versionado para skills: proposito, entradas, herramientas requeridas, permisos, dependencias, version y pruebas.
- Separar instrucciones de lenguaje natural de las autorizaciones ejecutables.
- Crear plantillas para agentes comunes: lectura documental, soporte interno y operaciones con aprobacion.
- Validar estructura, dependencias, permisos y pruebas antes de publicar un paquete.
- Establecer convenciones de versionado y compatibilidad entre runtime, skill y herramienta.

### Criterios de aceptacion

- Una skill no puede elevar permisos por su propio texto o metadata.
- Skills incompatibles o con dependencias ausentes no se cargan silenciosamente.
- Cada plantilla incluye pruebas y un ejemplo que funciona sin credenciales.
- Se puede actualizar o retirar una version con instrucciones claras.

### Entregable

Catalogo inicial pequeno de skills y plantillas documentadas, con proceso de publicacion.

## Hito 6 — CLI e integraciones

**Estado:** Pendiente
**Dependencia:** Hitos 1–5

### Trabajo

- Crear CLI para inicializar, validar, ejecutar y probar un agente.
- Generar proyectos desde plantillas versionadas, mostrando permisos solicitados.
- Elegir una primera integracion de desarrollo y demostrar instalacion, ejecucion y actualizacion.
- Agregar adaptadores adicionales solo cuando compartan un contrato probado y exista una necesidad clara.
- Documentar diferencias de capacidad por plataforma; no declarar compatibilidad equivalente sin pruebas.

### Criterios de aceptacion

- Una persona puede crear y ejecutar una demo desde una instalacion limpia siguiendo la guia.
- El instalador no lee ni modifica archivos fuera del alcance declarado sin consentimiento.
- Cada adaptador tiene una prueba smoke y una matriz de capacidades.
- Una integracion fallida no degrada los controles del runtime.

### Entregable

CLI estable y primer adaptador de plataforma con guia de instalacion y desinstalacion.

## Hito 7 — Distribucion y mantenimiento publico

**Estado:** Pendiente
**Dependencia:** Hitos 0–6

### Trabajo

- Definir versionado semantico, changelog, soporte de versiones y politica de deprecacion.
- Publicar guia de arquitectura, threat model, modelo de permisos y limites conocidos.
- Automatizar pruebas, paquetes, escaneo de dependencias y publicacion de releases.
- Crear ejemplos ejecutables y un camino claro de contribucion y reporte responsable de problemas de seguridad.
- Recoger feedback de usuarios antes de ampliar el numero de plataformas o componentes.

### Criterios de aceptacion

- Cada release enlaza commit, pruebas, cambios incompatibles y dependencias.
- Las instrucciones de instalacion y actualizacion se prueban desde cero.
- Existe una politica visible de soporte, vulnerabilidades y divulgacion responsable.
- Las issues distinguen fallos, solicitudes de funciones y riesgos de seguridad.

### Entregable

Primera release publica de la plataforma, con documentacion de operacion y contribucion.

## Secuencia sugerida

```text
Agente existente
  -> persistencia y evals de referencia
  -> runtime reutilizable
  -> politicas y registro de herramientas
  -> estado durable
  -> evaluacion/observabilidad compartida
  -> skills y plantillas
  -> CLI e integraciones
  -> release publica
```

Los hitos 2–4 tienen dependencias parciales y pueden solaparse con cuidado; no se debe publicar un registro de skills antes de definir el modelo de permisos.

## Primeros 90 dias

### Dias 1–30: evidencia y demo

- Completar pruebas reproducibles y una primera ejecucion de evals reales.
- Elegir casos de fallo que deben bloquear una release.
- Mostrar el flujo de aprobacion y rechazo en una demo.
- Definir el comportamiento esperado de persistencia e idempotencia.

### Dias 31–60: persistencia y contratos

- Implementar reanudacion durable de aprobaciones.
- Extraer los primeros contratos del runtime sin reescribir todo el agente.
- Crear un segundo agente pequeno que valide la reutilizacion.

### Dias 61–90: politicas y evaluacion

- Mover registro y permisos de herramientas a una interfaz comun.
- Anadir casos de evaluacion para entradas invalidas, denegacion y fallos de herramienta.
- Publicar una arquitectura corta y los resultados medidos, incluidos fallos.

Los dias son una guia de enfoque; el hito no se cierra por fecha si los criterios no se cumplen.

## Indicadores de avance

- Porcentaje de ejecuciones de evaluacion reproducibles por commit.
- Tasa de aprobaciones omitidas o acciones no autorizadas: objetivo cero en los casos criticos.
- Porcentaje de reanudaciones seguras tras reinicio y bajo concurrencia.
- Latencia, coste y tasa de error por proveedor y herramienta, sin datos personales.
- Tiempo para crear y probar un agente nuevo a partir de una plantilla.
- Numero de adaptadores con pruebas reales, no solo documentacion.

No usar una sola puntuacion agregada para afirmar que un agente es seguro o apto para produccion.

## Riesgos y mitigaciones

| Riesgo | Mitigacion |
|---|---|
| Abstraccion prematura | Extraer interfaces a partir de un segundo caso real |
| Confianza excesiva en evals de LLM | Combinar reglas deterministas, revisiones y evals versionadas |
| Acciones repetidas tras reintento | Idempotencia, claves de ejecucion y estado durable |
| Exposicion de datos en trazas | Redaccion por defecto y pruebas que inspeccionen telemetria |
| MCP remoto no confiable | Identidad, allowlist, contratos locales y permisos minimos |
| Alcance demasiado grande | Limitar el primer lanzamiento a un runtime, un almacen y una integracion |
| Compatibilidad inestable | Matriz probada y soporte explicito por version |

## Registro de cambios de estado

Agregar aqui fecha, hito, enlace a evidencia y responsable cada vez que un hito cambie de estado. No registrar como completado trabajo que solo este planificado.
