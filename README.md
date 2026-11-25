# Roadmap: plataforma de agentes empresariales

Plan incremental para evolucionar desde un agente operativo concreto hasta una plataforma modular para construir, asegurar, evaluar y distribuir agentes de IA.

**Punto de partida:** [ops-agent-mcp](https://github.com/mauricio-acuna/ops-agent-mcp), un agente de operaciones con LangGraph, herramientas MCP, aprobacion humana, auditoria y pruebas automatizadas.

**Estado:** plan inicial. Las etapas describen trabajo futuro; no se consideran completadas hasta que sus criterios de aceptacion se verifiquen.

## Vision

Crear una plataforma que permita a un equipo:

- definir agentes y flujos de trabajo sin acoplar la logica del negocio a un proveedor de modelos;
- conectar herramientas con permisos explicitos y contratos verificables;
- exigir aprobacion humana para acciones sensibles;
- persistir y reanudar ejecuciones de forma segura;
- medir calidad, seguridad, coste y latencia antes y despues de cada cambio;
- integrar el runtime en distintas herramientas de desarrollo;
- distribuir componentes versionados con documentacion y ejemplos reproducibles.

La prioridad es ofrecer pocos componentes confiables y compatibles entre si, no maximizar el numero de agentes o integraciones.

## Como seguir el plan

1. Leer [ROADMAP.md](ROADMAP.md) y elegir la siguiente etapa desbloqueada.
2. No empezar etapas que dependan de contratos o controles todavia inestables.
3. Registrar decisiones de arquitectura y cambios de alcance en issues o pull requests.
4. Marcar una etapa como terminada solo al cumplir todos sus criterios de aceptacion.
5. Publicar evidencia verificable: pruebas, resultados de evaluacion, instrucciones reproducibles y limites conocidos.

## Principios

- **Aprobacion y reglas fuera del modelo:** el LLM propone; codigo determinista autoriza y ejecuta.
- **Minimo privilegio:** herramientas, datos y permisos se conceden explicitamente.
- **Sin afirmaciones sin evidencia:** no publicar resultados de evaluacion o seguridad no medidos.
- **Proveedor reemplazable:** conservar contratos internos estables y probar proveedores por separado.
- **Seguridad desde el diseno:** validar entradas, salidas, permisos, secretos y limites de ejecucion.
- **Compatibilidad gradual:** cada modulo debe poder probarse y adoptarse sin instalar toda la plataforma.
- **Trazabilidad:** conservar una relacion clara entre requisito, implementacion y prueba.

## Hitos

| Hito | Resultado | Estado |
|---|---|---|
| 0. Consolidar el agente base | Evals reproducibles, persistencia de aprobaciones y demo | Pendiente |
| 1. Extraer un runtime reutilizable | Interfaces comunes para agentes, modelos y ejecuciones | Pendiente |
| 2. Centralizar politicas y herramientas | Registro MCP y autorizacion consistente | Pendiente |
| 3. Persistir estado y memoria | Reanudar ejecuciones con retencion y aislamiento definidos | Pendiente |
| 4. Crear calidad y observabilidad | Evals, trazas y gates de CI | Pendiente |
| 5. Distribuir skills y plantillas | Componentes versionados y verificables | Pendiente |
| 6. Agregar CLI e integraciones | Instalacion y adaptadores con capacidades declaradas | Pendiente |
| 7. Preparar una version publica | Documentacion, compatibilidad y proceso de releases | Pendiente |

El detalle, las dependencias y los criterios de terminado estan en [ROADMAP.md](ROADMAP.md).

## Fuera del alcance inicial

- Entrenar modelos fundacionales propios.
- Permitir ejecucion arbitraria de comandos o herramientas por defecto.
- Prometer proteccion perfecta contra prompt injection.
- Replicar todas las funciones de otros productos o soportar todos los entornos a la vez.
- Publicar cifras de calidad, coste o rendimiento sin datos reproducibles.

## Estimacion orientativa

Para una persona desarrollando junto con un empleo, considerar **12–18 meses** para recorrer las etapas principales con alcance controlado. Es una estimacion de planificacion, no una garantia; los hitos se gobiernan por evidencia y no por calendario. Una primera version util puede limitarse a los hitos 0–4.
