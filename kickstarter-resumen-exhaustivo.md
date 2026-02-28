# Sesion de Diseno: Sistema Agentico de Desarrollo de Software

**Fecha:** 28 de febrero de 2026
**Duracion:** 54 minutos
**Participantes:** 3 personas (Silvetti/Cilveti, Isma, Carlos)

---

## 1. Punto de partida de la conversacion

La sesion arranca retomando una discusion previa sobre un loop basico: un agente desarrollador coge una epica, ejecuta las tareas y termina. El debate inicial es: **que pasa cuando termina una epica?** Silvetti plantea que no deberia arrancar automaticamente otro loop de desarrollo, sino que deberian existir agentes especializados intermedios que evaluen el resultado antes de continuar.

Se menciona como referencia a "un tipo" (de un articulo/hilo de Twitter) que habia montado un sistema similar donde:
- A las 9-10 de la manana lanzaba un proceso de priorizacion de tareas
- 2 horas despues lanzaba un agente de smoke test
- Luego lanzaba un agente arquitecto
- **Su fuente de la verdad era su panel** (tablero de tareas), y todos los agentes operaban en base a el

Se reconoce que la historia de este tipo esta escrita a posteriori y no se sabe cuanto tenia previamente construido.

---

## 2. Debate: Abstracto vs. Concreto

Se plantea una decision fundamental: **Queremos crear una maquina generica que cree cualquier SaaS, o una maquina que cree un SaaS concreto?**

- Lo abstracto obliga a pensar en piezas genericas reutilizables
- Lo concreto da mas conocimiento practico de las necesidades reales

**Decision tomada:** Crear una maquina "lo suficientemente generica para lo que necesitemos". No totalmente abstracta, pero tampoco atada a un unico proyecto. Se va a usar un proyecto concreto (parece ser algo de "localizar gente para jugar a rol") como banco de pruebas.

Se valora que el MVP del proyecto piloto no es excesivamente grande. No se podria hacer en una sola sesion de Claude Code, pero tampoco es algo descomunal. Tiene partes moviles pero es abordable.

---

## 3. Descarte de hooks de Claude Code

Se discute el uso de los hooks nativos de Claude Code (pre/post sesion, herramientas, etc.). **Se descartan** por las siguientes razones:

- Los hooks de Claude Code son **deterministas**, no intencionales. Se disparan cuando acaba la sesion, cuando se llama una herramienta, etc. No tienen "inteligencia"
- El comando `/exit` de Claude Code **pierde el contexto** de la sesion, lo cual es un problema critico
- Se necesita un control mas fino y deliberado sobre que triggerea el siguiente paso

**Decision:** No usar hooks de Claude Code como mecanismo de orquestacion. La orquestacion debe vivir en el panel/sistema de tareas.

---

## 4. El Panel como fuente de la verdad - Consenso principal

**Consenso unanime:** El panel (conjunto de epicas y tareas) es la fuente de la verdad unica y absoluta del proyecto. Todo gira en torno a el.

Razones:
- Es un **a priori**: se define antes de que la maquina empiece a trabajar
- Da **visibilidad** a los humanos sobre que esta haciendo la maquina
- Permite saber de antemano que loops se van a generar
- Permite que con "cierta inteligencia" (del orquestador) se decida a que epica ir segun el resultado

### El panel como observatorio (modo heroe)

Se establece una regla tipo videojuego: **los humanos no pueden tocar el panel una vez arrancado**. Solo observan.

- "Modo heroe": si la maquina muere, se para todo y se echa para atras. No hay saves intermedios editables por humanos
- Los humanos ven el panel para monitorizar, pero no interfieren
- Si ven que se ha "petado", la opcion es parar y rebobinar, no intervenir manualmente
- Se plantea brevemente la idea de un chat con un agente que pudiera tocar el panel, pero se descarta para mantener la pureza del experimento

**Motivacion:** Lo que se quiere comprobar es si se puede crear una maquina casi totalmente automatica. La gracia esta en no tocar.

---

## 5. Beats como sistema de gestion de tareas

### Que es Beats

- Es un **CLI de gestion de tareas** (no es inteligente por si mismo)
- Software libre, evolucion de conceptos que uno de los participantes habia montado previamente en Obsidian
- Soporta: epicas, tareas, subtareas, dependencias entre tareas, estados (open, in progress, done, etc.), prioridades
- Usa una **base de datos basada en Git** (concepto similar a Git para tracking)
- Utiliza el concepto de **worktree**: crea snapshots del estado del proyecto para poder comparar y mergear en terminos de conocimiento (no de Git)
- Tiene un hook de preinicio de sesion de Claude Code que inyecta contexto (tareas pendientes, estado actual)
- El agente puede lanzar comandos del CLI para consultar epicas, tareas, etc.

### Agente de descomposicion existente

Uno de los participantes ya tiene un agente que:
- Coge un plan/especificacion
- Extrae epicas, subtareas, dependencias
- Identifica que se puede hacer en paralelo
- Usa el CLI de Beats por debajo

Este agente podria reutilizarse y mejorarse para el proyecto.

### Integracion Beats + Claude Code

- Al arrancar una sesion de Claude Code, el hook de inicio de sesion de Beats lanza un primer comando que carga el contexto (tareas pendientes, estado)
- El agente puede preguntar "en que punto estabamos?" y Beats le informa

---

## 6. Panel vs. Beats: donde poner la logica

**Debate extenso** sobre si la logica de triggers/hooks deberia vivir en Beats (capa de datos) o en el Panel (capa de visualizacion).

### Argumentos para ponerla en Beats:
- Beats es el sistema que gestiona las tareas, tiene sentido que el sea quien dispare cosas
- Evita confusiones de tener dos fuentes de verdad (Beats + Panel modificando por encima)
- Es mas coherente arquitectonicamente

### Argumentos en contra de tocar Beats:
- Es codigo open source complejo, "no es la cosa mas sencilla de tocar"
- Riesgo de crear un fork que no se pueda actualizar
- Si se atan a una version modificada, pierden la capacidad de actualizarse

### Experiencia previa con paneles:
- **Primer panel probado (Kanban):** solo lectura, no dejaba modificar ni crear tareas. Era un desastre
- **Segundo panel probado:** interactivo, permitia crear/borrar/mover tareas. Usaba Beats por debajo. Pero la interaccion no se va a usar (modo heroe)

### Decision:
- **El panel lee de Beats** para mostrar las tarjetas/estado
- **Los triggers idealmente deberian vivir en Beats** (mas coherente)
- Hay que investigar si Beats ya tiene hooks nativos
- Si no los tiene, evaluar cuanto cuesta anadirlos solo para este proyecto
- Alternativa: si modificar Beats es demasiado complejo, poner los triggers en la capa del panel que opera sobre Beats

---

## 7. Sistema de triggers: que eventos disparan que

Se acuerda que se necesita un sistema de triggers para los siguientes eventos:

| Evento | Accion esperada |
|--------|----------------|
| Una tarea cambia de estado | El sistema sabe que puede continuar con la siguiente |
| Se crean nuevas tareas | Se registran en el panel |
| Se editan tareas | Se actualiza el panel |
| Una epica termina completa | Se despierta el Arquitecto |
| Una sesion se "rompe"/cicla | Se despierta el Arquitecto |
| Periodicamente (cron) | Se despierta el Arquitecto para chequear estado |

### Problemas de dependencias entre tareas:
- Si una tarea depende de otra y se intenta cerrar fuera de orden, que pasa?
- Beats ya tiene algun hook que avisa si intentas cerrar una tarea cuando hay otra en "in progress"
- Hay que refinar esta logica

---

## 8. Un loop = Una epica

**Decision importante:** Una sesion de Claude Code se encarga de una epica entera (con todas sus subtareas).

Flujo:
1. La epica pasa a "to do" con todas sus subtareas
2. La sesion de Claude Code va moviendo cada subtarea a "in progress" y luego a "done"
3. La epica en si esta "in progress" mientras haya subtareas pendientes
4. Cuando todas las subtareas estan "done", la epica esta completa
5. El agente ejecutor debe tener las herramientas para validar su propio trabajo (levantar navegador, hacer peticiones, etc.)
6. **Se confia plenamente en el agente ejecutor** para el trabajo dentro de la epica. No se pone una segunda capa de revision dentro del loop

### Hipotesis a validar:
- "Un loop resuelve una epica" es una hipotesis que quieren comprobar

---

## 9. El Arquitecto (antes llamado "Orquestador Revisor")

Esta es la pieza mas discutida y la que genera mas entusiasmo. Se renombra de "agente orquestador revisor" a **"El Arquitecto"** (referencia a Matrix).

### Que es el Arquitecto

- Es la **inteligencia suprema** del proyecto
- Es el unico agente con capacidad de tomar decisiones a nivel de proyecto
- Es **proactivo** (como el concepto de "soul" de OpenAI/Open Cloud)
- No esta corriendo todo el rato. Se despierta, absorbe contexto (alma + intenciones + memoria) y toma decisiones
- Es el equivalente a un Product Owner inteligente

### Cuando se activa

Dos mecanismos complementarios:
1. **Por trigger:** Cuando una epica termina, se dispara el Arquitecto
2. **Por cron:** Periodicamente (cada X minutos) se despierta para chequear el estado general del sistema

El cron es importante porque:
- No depende de que termine una tarea para actuar
- Puede detectar sesiones rotas o cicladas
- Genera una timeline de actividad ("a las 9 se desperto y vio X, a las 9:30 se desperto y vio Y")

### Que hace cuando se despierta

1. Revisa el estado del proyecto
2. Revisa si hay alguna epica recien completada
3. Genera un resumen de si la epica termino como se habia definido o hay ajustes
4. Analiza la siguiente epica planificada
5. Decide si tiene sentido arrancar con esa o con otra
6. Puede **crear nuevas epicas** que no existian en el plan original
7. Puede **reordenar prioridades**
8. Puede detectar necesidad de refactors y proponerlos como epicas
9. Puede detectar bugs y priorizarlos
10. Arranca la siguiente sesion de desarrollo

### Que NO hace

- No toca codigo directamente
- No revisa si el codigo de una tarea funciona (eso es responsabilidad del agente ejecutor)
- Lo que revisa es coherencia a nivel de proyecto: la epica terminada permite continuar con la siguiente?

### Restriccion critica: SOLO UNA INSTANCIA

**Debe estar garantizado que jamas haya mas de una instancia del Arquitecto corriendo.** Esto es critico porque:
- Multiples instancias podrian tomar decisiones contradictorias
- Podria haber un trigger de epica completada + un cron coincidiendo
- Hay que implementar logica de "ya hay uno corriendo? entonces espero o me duermo"

### Como arranca nuevas sesiones

**Debate sobre implementacion:**
- El Arquitecto necesita poder lanzar nuevas sesiones de Claude Code para ejecutar epicas
- Cada sesion de Claude Code es un proceso/terminal
- Se puede lanzar como un script (proceso del SO) sin necesidad de abrir una terminal visible
- Se puede matar con kill -9 del PID si es necesario

**Decision:** El Arquitecto no debe lanzar CLIs directamente. Debe tener **tools abstractas** especificas:
- `lanzar_nueva_epica` -> por debajo levanta un script que inicia una sesion de Claude Code con el prompt adecuado
- El Arquitecto no sabe (ni necesita saber) que hay por debajo de esa tool
- Esto previene el caos de que el Arquitecto lance 7 sesiones de Claude Code arbitrariamente

### Composicion del Arquitecto

En terminos de contexto, el Arquitecto es:
- Un **alma** (intencion del proyecto, intent engineering)
- Unas **intenciones** (que se quiere lograr)
- Una **memoria** (historial de decisiones, estado acumulado)
- Un set de **tools** (lanzar epica, consultar Beats, etc.)

### Debate: debe mirar codigo?

- No esta claro si el Arquitecto debe tener acceso a explorar el codigo
- Alternativa: que tenga un subagente explorador que le reporte
- Lo que si esta claro es que su funcion principal es **tomar decisiones**, no revisar codigo

---

## 10. Especificacion e intencion: dos capas

Se debate extensamente que pasa cuando la maquina, durante su ejecucion, necesita cambiar la especificacion original del proyecto.

### Dos niveles de "contrato":

1. **Alma / Intencion (inmutable):** El proposito fundamental del proyecto. Por que existe. Que resultado se busca (ej: "obtener buenos resultados economicos"). Esto **nunca se puede cambiar** automaticamente
2. **Especificaciones (mutables por el Arquitecto):** El como se implementa. Que features tiene, como estan estructuradas. El Arquitecto **puede** decidir cambiarlas

### Proteccion contra derivas

- Riesgo identificado: que los loops vayan cambiando cosas y la fuente de la verdad se aleje de la idea original sin que los humanos lo quieran
- Solucion: la maquina no tiene acceso a tocar la raiz (intencion/alma)
- Si el Arquitecto necesita cambiar algo fundamental, debe:
  - Documentarlo
  - Marcarlo en rojo en el panel (senal visible para los humanos)
  - Generar una alarma tipo "necesito un humano" o al menos dejar constancia de que a partir de ese momento se ha tomado una decision que rompe con el contrato inicial

### Concepto de "nueva linea temporal"

Cuando el Arquitecto toma una decision que diverge del plan original, seria como una "bifurcacion en el espacio-tiempo". El panel deberia mostrar visualmente que a partir de ese punto se esta en una nueva linea temporal.

---

## 11. Mecanismo de rollback

Se discute que pasa cuando la maquina llega a un callejon sin salida.

### Usando Git + Beats

- Beats usa una base de datos basada en Git
- Se podria "rebobinar" tanto el estado del panel como el estado del codigo a un punto anterior
- Concepto de **checkpoint**: cuando una epica termina correctamente, se hace un checkpoint (commit)
- Si hay que volver atras, se vuelve al ultimo checkpoint valido

### Usando Worktrees

- Beats usa el concepto de worktree de Git
- Crea snapshots del momento actual antes de empezar una tarea
- Se puede comparar el estado antes y despues
- Se puede mergear "en terminos de conocimiento" (no de Git puro)

### Dudas sin resolver:

- El Beats estaba en `.gitignore` del proyecto. Como se coordina el rollback del proyecto con el rollback del panel si viven en sitios distintos?
- Pueden aparecer problemas recursivos si se intenta deshacer cambios que ya habian modificado el contrato inicial
- Hay que investigar como funciona el sistema de worktrees en detalle

---

## 12. Flujo completo acordado

```
[HUMANO] -> Crea especificacion + alma del proyecto
         -> Agente descompone en epicas/tareas/dependencias (Beats)
         -> Humano pulsa boton "INICIAR"
                    |
                    v
[AGENTE EJECUTOR] -> Sesion de Claude Code
                  -> Coge primera epica
                  -> Ejecuta todas las subtareas secuencialmente
                  -> Valida su propio trabajo (navegador, tests, etc.)
                  -> Marca epica como completada en Beats
                    |
                    v
[TRIGGER] -> Epica completada dispara al Arquitecto
                    |
                    v
[EL ARQUITECTO] -> Se despierta
                -> Lee alma + intenciones + estado del proyecto
                -> Revisa resultado de la epica completada
                -> Evalua: termino como se esperaba?
                -> Decide: cual es la siguiente epica?
                   - Puede ser la que estaba planificada
                   - Puede reordenar
                   - Puede crear epicas nuevas
                -> Si necesita cambiar especificaciones -> Senal roja en panel
                -> Lanza nueva sesion (tool: lanzar_nueva_epica)
                -> Se duerme
                    |
                    v
[AGENTE EJECUTOR] -> Nueva sesion para la siguiente epica
                  -> (ciclo se repite)

[CRON] -> Cada X minutos despierta al Arquitecto
       -> Chequea si hay sesiones rotas/cicladas
       -> Si hay algo que atender -> actua
       -> Si todo va bien -> se duerme
```

---

## 13. Referencias tecnologicas a investigar

| Referencia | Para que |
|-----------|---------|
| **OpenHands (antes OpenDevin)** | Como gestionan los loops de agentes, orquestacion de sesiones |
| **Hermes** | Version simplificada de OpenHands, puede dar patrones mas limpios para el loop |
| **OpenFan** | Relacionado con OpenHands, para simplificacion |
| **Concepto "Soul" de Open Cloud / OpenAI** | Para el diseno del caracter proactivo del Arquitecto |
| **Worktrees de Git** | Para el mecanismo de snapshots y rollback |
| **Beats CLI** | Investigar si tiene hooks nativos, evaluar coste de anadirlos |

---

## 14. Dudas abiertas sin resolver

### Sobre Beats y el sistema de triggers
1. **Tiene Beats hooks nativos?** Hay que investigarlo. Si los tiene, se usan directamente. Si no, hay que evaluar cuanto cuesta anadirlos
2. **Donde viven los triggers?** En Beats (capa datos) o en el Panel (capa vista)? Hay argumentos para ambos lados, aunque se inclina por Beats
3. **Que pasa con las dependencias cuando una tarea se cierra fuera de orden?** Beats parece tener algo pero hay que refinarlo
4. **Merece la pena hacer un fork de Beats?** Riesgo de no poder actualizar vs. necesidad de customizacion
5. **Como se coordina el rollback de Beats (gitignore) con el rollback del proyecto (Git)?**

### Sobre el Arquitecto
6. **Cuanta agencia le damos?** Puede cambiar especificaciones, pero hasta donde? Puede crear epicas ilimitadamente?
7. **Debe mirar codigo directamente o tener un subagente explorador?**
8. **Como se implementa la restriccion de instancia unica?** Lock file? PID check? Mutex?
9. **Que pasa si el Arquitecto se cicla a si mismo?** Necesita deteccion de ciclos propia
10. **Donde vive el script del Arquitecto?** Si esta dentro del mismo proyecto donde esta el Beats, como se levanta sin conflictos?
11. **Cual es la mejor forma de levantar el script?** Proceso en background, tmux, screen, servicio del SO?

### Sobre las sesiones de Claude Code
12. **Como lanza el Arquitecto una nueva sesion de Claude Code?** Script que ejecuta el CLI? Como se pasa el prompt/contexto?
13. **Como se evita que las sesiones se interfieran entre si?** Si el Arquitecto lanza multiples, pueden pisar el mismo codigo
14. **Que pasa cuando una sesion de Claude Code se rompe?** Como se detecta y como se recupera?

### Sobre el flujo general
15. **Que ocurre cuando el Arquitecto decide crear una epica nueva que no existia?** Como afecta al plan general? Como se notifica?
16. **Como se visualizan las "lineas temporales" divergentes en el panel?**
17. **Que herramientas concretas necesita el Arquitecto como tools?** (lanzar_epica, consultar_estado, crear_epica, modificar_especificacion, alarma_humano, ...)
18. **Hace falta un agente "inicializador" separado o el boton humano es suficiente?**

---

## 15. Tareas concretas a realizar

### Investigacion
| # | Tarea | Responsable | Notas |
|---|-------|-------------|-------|
| 1 | Investigar si Beats tiene hooks/triggers nativos | Por asignar | Si los tiene, usarlos. Si no, evaluar coste de anadirlos |
| 2 | Investigar OpenHands y Hermes | Por asignar | Foco en como gestionan loops, orquestacion, deteccion de ciclos |
| 3 | Investigar concepto de worktree en Beats | Por asignar | Como funciona el snapshot, como se compara, como se mergea |
| 4 | Investigar como lanzar sesiones de Claude Code como procesos | Por asignar | Scripts, PIDs, gestion de multiples sesiones |

### Diseno
| # | Tarea | Responsable | Notas |
|---|-------|-------------|-------|
| 5 | Definir el alma/intencion del proyecto piloto | Los 3 | Documento inmutable que sera la raiz del proyecto |
| 6 | Crear especificacion/PRD del proyecto piloto | Los 3 | Partir de esta transcripcion, destilar en especificaciones |
| 7 | Disenar el Arquitecto: prompt, tools, ciclo de vida | Por asignar | Alma + intenciones + memoria + set de herramientas |
| 8 | Disenar el sistema de triggers/eventos | Por asignar | Que eventos -> que acciones, tabla completa |
| 9 | Disenar el panel de visualizacion | Por asignar | Lee de Beats, modo solo lectura, senales rojas, timeline |
| 10 | Definir el mecanismo de instancia unica del Arquitecto | Por asignar | Garantizar que nunca hay 2 corriendo |

### Construccion
| # | Tarea | Responsable | Notas |
|---|-------|-------------|-------|
| 11 | Modificar/extender Beats para hooks (si necesario) | Por asignar | Solo lo minimo necesario |
| 12 | Crear las tools del Arquitecto | Por asignar | lanzar_epica, consultar_estado, crear_epica, etc. |
| 13 | Crear script de arranque del Arquitecto | Por asignar | Con cron y con trigger |
| 14 | Crear el agente de descomposicion plan->epicas->tareas | Por asignar | Mejorar el que ya existe |
| 15 | Crear el panel de visualizacion | Por asignar | Sobre Beats, solo lectura |
| 16 | Implementar mecanismo de rollback | Por asignar | Checkpoint + vuelta atras coordinada |

---

## 16. Principios de diseno acordados

1. **El panel (Beats) es la unica fuente de la verdad**
2. **Los humanos no intervienen una vez arrancado** (modo heroe)
3. **Un loop = una epica = una sesion de Claude Code**
4. **El agente ejecutor es responsable de validar su propio trabajo**
5. **Solo hay UN Arquitecto, nunca multiples instancias**
6. **El alma/intencion del proyecto es inmutable**
7. **Las especificaciones pueden cambiar, pero se documenta y senaliza**
8. **El Arquitecto trabaja con tools abstractas, no con CLIs directos**
9. **El Arquitecto es proactivo (cron) y reactivo (triggers)**
10. **Lo que queremos comprobar es si podemos encadenar loops en una maquina de loops** - eso es un sistema agentico puro

---

## 17. Frase de cierre

> "Creo que igual tiene mas sentido que intentemos arrancar y nos encontremos con las cosas que vamos viendo."

La sesion cierra con la sensacion de que hay suficiente base teorica para empezar a construir y que seguir teorizando sin implementar tiene rendimientos decrecientes. Siguiente paso: transcribir esta sesion (hecho), destilar un PRD, y empezar a construir.
