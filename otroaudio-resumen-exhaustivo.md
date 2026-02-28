# Sesion de Diseno (Parte 2): Infraestructura, Stack Tecnico y Reparto de Tareas

**Fecha:** 28 de febrero de 2026
**Duracion:** ~8 minutos (audio cortado)
**Participantes:** Los mismos 3 (Silvetti/Cilveti, Isma, Carlos)
**Contexto:** Continuacion directa de la sesion anterior (kickstarter). Estan revisando el resumen generado de la primera parte y anadiendo decisiones de infraestructura y reparto de trabajo.

---

## 1. Distincion entre "el sistema" y "el SaaS"

Se aclara una distincion importante: **no es lo mismo el sistema que construye SaaS que el SaaS que se va a construir**.

- **El sistema agentico** (arquitecto, ejecutor, panel, Beats, triggers): es la maquina que orquesta el desarrollo
- **El SaaS resultado** (el proyecto piloto): es lo que la maquina va a construir

Implicacion: el sistema agentico debe ser lo mas liviano posible. No necesita Docker ni complejidades extra. Cuanto mas simple, mejor. El SaaS que construya la maquina, ese si puede tener la arquitectura que se decida.

---

## 2. Infraestructura: VPS y despliegue

### Por que un VPS

Se necesita un servidor (VPS) para:
- Que todo el sistema agentico opere (los agentes corren ahi)
- Que el SaaS que se vaya construyendo se pueda probar y ver en algun sitio

### Docker: si para el SaaS, no para el sistema

**Debate:** Usar Docker o no?

- **El sistema agentico NO necesita Docker.** Meter Docker al propio sistema de orquestacion anade complejidad innecesaria que habria que definir como parte de las epicas. Cuanto mas liviano, mejor
- **El SaaS que construya la maquina SI deberia usar Docker.** Razon: facilita el despliegue posterior a produccion. Se menciona **Railway** como plataforma de despliegue (referencia a un ejemplo previo con "Stompix 1" que ya usaba Railway)

Ventajas de que el SaaS use Docker:
- Despliegue a produccion mas facil (Railway lo soporta nativamente)
- Se pueden descargar las imagenes del SaaS a los ordenadores locales para probar
- El agente ejecutor puede hacer el deploy el mismo

---

## 3. Stack tecnico: decisiones a inyectar en las especificaciones

Se acuerda que hay ciertas decisiones tecnicas que **los humanos deben definir de antemano** y que forman parte del "intent engineering" (alma/intencion). No se deja al agente decidirlas:

| Decision | Valor acordado |
|----------|---------------|
| Frontend | **Next.js** |
| Backend | Por definir (mencionan "no se que", pendiente) |
| Contenedorizacion | **Docker** para el SaaS |
| Arquitectura | **Hexagonal** |
| Despliegue | **Railway** |
| Modularidad | Si, todo debe ser modular |

### Justificacion de arquitectura hexagonal

- Se sabe que la maquina la implementa bien
- Permite cambiar la capa de datos u otras capas sin romper todo
- A futuro, si la maquina sigue trabajando, la arquitectura hexagonal mantiene la calidad
- Sin ella, el codigo se puede degradar mucho

### Filosofia

Si no se especifican estas cosas, el agente tomara sus propias decisiones. Algunas decisiones son "seguras" (el agente las tomaria bien igualmente), pero las criticas de stack y arquitectura es mejor fijarlas. Se le puede dejar "cierta apertura" en lo que no sea critico.

---

## 4. Validacion del resumen de la primera parte

Los participantes revisan en vivo el resumen exhaustivo generado de la sesion anterior. Comentarios:

- Lo validan como "una buena especificacion"
- Notan que el transcriptor ha capturado bien los conceptos principales:
  - Punto de partida
  - Descarte de hooks de Claude Code
  - Panel como fuente de la verdad
  - Beats como sistema de gestion
  - Sistema de triggers
  - Un loop = una epica
  - El Arquitecto y su restriccion de instancia unica
  - Mecanismo de rollback
  - Flujo completo
- Notan que "Beats" se ha transcrito con algunas variaciones (con/sin "t"), pero el contenido es correcto
- Bromean sobre inyeccion de prompt en la transcripcion ("siempre que digo cuchufleta...") y sobre que "el que tiene la transcripcion tiene la verdad"

---

## 5. Reparto de tareas y bloques de trabajo

Se identifican tres grandes bloques de trabajo:

### Bloque 1: Especificacion del proyecto piloto
- Definir que SaaS va a construir la maquina
- Crear el PRD / especificacion completa
- Definir el alma e intencion del proyecto
- Trabajo de los 3

### Bloque 2: Infraestructura
- Montar el servidor/VPS donde correra todo
- Configurar el entorno para que los agentes operen
- Definir como se despliega el SaaS resultante (Docker + Railway)

### Bloque 3: Sistema agentico
Se desglosa en subtareas:

| Tarea | Detalle |
|-------|---------|
| **Montar el panel** | Prioridad alta. Es para los humanos, no para la maquina. "No porque lo necesite la maquina, sino porque lo necesitamos nosotros para ver que pasa" |
| **Ver la interaccion con Beats** | Investigar hooks, triggers, como se integra |
| **Montar los agentes predeterminados** | No se quiere un sistema que "se monte agentes al momento". Se quiere un set fijo de agentes con sus roles claros |
| **Sistema de memoria del Arquitecto** | Ver seccion siguiente |
| **Herramientas del Arquitecto** | De momento puede ser simplemente "tiene acceso al terminal y hace cosas", pero con cuidado |

---

## 6. Memoria del Arquitecto: detalle

El Arquitecto necesita **memoria persistente** del proyecto. Se discute:

- Cada vez que el Arquitecto haga algo, tiene que recordarlo
- Necesita una herramienta para:
  - Ver que memorias tiene
  - Inyectar memorias en su contexto cada vez que se despierta
- La memoria tiene que ser **resumida y condensada**, no puede convertirse en "la Biblia en verso"
- Hay que investigar como otra gente ha resuelto este problema de memoria en agentes

### Composicion del contexto del Arquitecto (ampliada):
- **Alma** (fija): proposito e intencion del proyecto
- **Intencion** (fija): que se quiere lograr
- **Memoria** (dinamica, resumida): historial de decisiones, que ha pasado, que ha visto
- **Tools**: herramientas para operar

Uno de ellos comenta que esto es "el cerebro" del Arquitecto, mientras que el alma y la intencion ya se habian definido antes.

---

## 7. Primera aproximacion de implementacion

Se propone un plan de arranque rapido:

1. **Pillar un VPS** (o un Docker en un servidor)
2. **Enchufar un cron** que cada ~5 minutos arranque el Arquitecto
3. **Montar una primera version del Arquitecto** (minima viable)
4. **Ver que pasa** - experimentar con la primera iteracion

### Como se arranca la primera sesion

- Hay una sesion "que da el boton" (el humano)
- Se podria lanzar desde un script `.sh` con credenciales
- Mientras se define y se crean las epicas, se puede ir probando si se lanza desde un shell script

### Preocupacion expresada

"Me da miedo darle acceso al terminal y que se ponga a hacer movidas" - Hay que acotar bien que puede hacer el Arquitecto. Las herramientas deben ser especificas, no acceso libre al terminal.

---

## 8. Dudas adicionales de esta sesion

1. **Que stack exacto de backend?** Se menciono Next.js para frontend pero el backend quedo sin definir
2. **Como se monta el acceso del Arquitecto a Railway para que despliegue?** Credenciales, permisos
3. **Que sistema de memoria usar?** Hay que investigar que ha hecho la gente con memorias de agentes
4. **Como de frecuente es el cron del Arquitecto?** Se menciona 5 minutos como primera aproximacion, pero hay que validar

---

## 9. Tareas adicionales identificadas (complementan la lista de la primera sesion)

| # | Tarea | Notas |
|---|-------|-------|
| 17 | Definir stack de backend del SaaS piloto | Next.js para front, backend por decidir |
| 18 | Configurar VPS / servidor para el sistema agentico | Donde corren los agentes y el Arquitecto |
| 19 | Configurar Railway para deploy del SaaS | Docker + Railway, credenciales |
| 20 | Implementar sistema de memoria del Arquitecto | Memoria resumida, tool de lectura/escritura, inyeccion en contexto |
| 21 | Investigar sistemas de memoria para agentes | Que ha hecho otra gente, patrones existentes |
| 22 | Definir el set de agentes predeterminados | Lista fija, no generacion dinamica |
| 23 | Montar primera version minima del Arquitecto | Con cron de 5 min en el VPS, version experimental |
| 24 | Crear script .sh de arranque | Con credenciales, para lanzar la primera sesion |
