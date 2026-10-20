# ORBIT — De una petición operativa a un resultado comprobado

**César Outón Leiro · Automatización con IA y operaciones IT · Octubre de 2026**

Proyecto personal · Varias semanas y más de 100 horas de trabajo (estimación propia).

Cuando una ausencia deja una guardia sin cubrir, encontrar a alguien disponible no resuelve el problema. Hay que comprobar su cualificación, descanso y coste, coordinar varios sistemas y confirmar que los cambios quedaron aplicados. Durante una caída, cerrar el ticket tampoco demuestra que el servicio vuelva a funcionar.

Diseñé Orbit para abordar esa coordinación: un asistente de IA investiga y propone soluciones; reglas controladas por la empresa determinan qué puede cambiar; una comprobación posterior confirma el resultado. El beneficio buscado es reducir seguimiento manual, acciones duplicadas y trabajos cerrados antes de tiempo.

**Este caso de diseño utiliza dos recorridos simulados y datos ficticios.** Presenta mi enfoque técnico y operativo, sin atribuir resultados de producción a la simulación.

## Mi aportación

Dirigí la definición operativa y la iteración del proyecto con herramientas de IA. Convertí las necesidades en tres conjuntos de artefactos: reglas de autoridad y cobertura, contratos entre sistemas y criterios de evaluación. Mi responsabilidad fue decidir qué información hacía falta, qué riesgos debían bloquear un cambio y qué prueba permitiría darlo por completado.

Tres decisiones articulan la solución: mantener los permisos fuera del modelo, conservar explícitamente la incertidumbre y verificar el servicio de forma separada del ticket. La construcción de código fue asistida por IA; mi contribución se centró en dirigir, revisar y dar coherencia al conjunto.

## Una decisión concreta: recuperar cobertura

El coordinador de cobertura recibe ausencias autorizadas que dejan turnos futuros sin cubrir. El primer sustituto disponible no cumple el descanso mínimo. Mover a otra persona abriría un hueco distinto. Existe apoyo externo, pero consume un presupuesto limitado.

Orbit consulta calendario, personas, turnos y proveedores. Compara alternativas válidas y explica los descartes. Una opción barata no es aceptable si incumple descanso; una opción técnicamente posible tampoco puede ejecutarse sin autoridad para comprometer su coste.

El resultado esperado es cobertura comprobada dentro de las restricciones. No se fija de antemano cuántas contrataciones debe realizar ni qué herramientas debe consultar primero.

Si otro coordinador modifica una guardia durante el proceso, la operación dependiente debe rechazarse cuando exista control de versiones. Orbit actualiza la información y replantea. Si solo una parte del plan se ha aplicado, identifica esos efectos y comprueba si puede completar o compensar lo pendiente sin crear un nuevo riesgo. No reinicia todo a ciegas.

## Cómo se reparte la responsabilidad

**Entender la petición → investigar y proponer → autorizar y aplicar → comprobar e informar.**

La empresa define reglas y permisos. Orbit decide cómo investigar dentro de ellos. Los conectores consultan o modifican los sistemas existentes; un control externo permite o rechaza cada cambio. El verificador consulta después el estado resultante.

Por ejemplo, el conector lee una guardia, la política rechaza asignar a alguien de vacaciones y el verificador comprueba el calendario actualizado. Son responsabilidades diferenciadas; no requieren una aplicación independiente para cada bloque.

El registro conserva la petición, la autorización, las acciones y sus comprobaciones. Un revisor debe poder reconstruir qué ocurrió y qué quedó pendiente.

## Otro problema: recuperar un servicio crítico

El operador de incidentes recibe una alerta de caída, clasificada como incidente crítico o P1. El ticket contiene síntomas, pero no una causa ni una solución.

En el ejemplo, el tráfico llega a una puerta distinta de aquella donde escucha la aplicación: la configuración apunta al puerto 8081 y el proceso utiliza el 8080. Orbit contrasta esa diferencia con logs y cambios recientes. Compara corregir el destino, desviar tráfico y revertir el despliegue.

La elección depende del riesgo comprobable y de los permisos. Un cambio pequeño no siempre es el más seguro; una reversión conocida puede ser preferible si la evidencia es insuficiente. Cuando la discrepancia está confirmada, corregirla permite conservar el resto del despliegue.

Después se comprueban el servicio, su acceso externo y la configuración relevante durante una ventana definida. Si el proveedor acepta el cambio pero la caída continúa, el incidente permanece abierto. La urgencia modifica las reglas de respuesta aplicables; no concede autoridad adicional.

## Decisiones técnicas que sostienen el diseño

| Decisión | Valor | Compromiso |
|---|---|---|
| IA para investigar; reglas externas para autorizar | Flexibilidad sin delegar permisos al modelo | Mantener políticas y contratos comprobables |
| Verificar después de cambiar | Detectar aceptación sin efecto y falsos cierres | Más consultas y una ventana de observación |
| Reservar presupuesto antes de comprometer gasto | Evitar que dos operaciones consuman el mismo saldo | Registro presupuestario compartido y reserva atómica |
| Revalidar el estado antes de escribir | Detectar cambios ajenos y datos obsoletos | Depender de las garantías de cada API |

La reserva presupuestaria cuenta compromisos confirmados y reservas pendientes; un timeout no libera fondos automáticamente. Si otros actores pueden gastar fuera de ese control, se debe reconciliar su actividad antes de prometer un límite global.

Cuando una API no permite escrituras condicionadas por versión, una pasarela puede coordinar los escritores que la utilizan. Si no puede excluir cambios externos y la operación resulta insegura, bloquea la automatización y deriva a una vía controlada. Leer justo antes de escribir no elimina por sí solo esa carrera.

## Extender la solución

Cobertura e incidentes son dos usos propuestos del diseño. Incorporar empleados o gestionar dispositivos reutilizaría investigación, control y verificación; cada dominio necesita sus propios conectores validados, políticas y criterios de éxito. Compartir un núcleo no significa disponer automáticamente de esos permisos o integraciones.

## Cómo evaluaría calidad y valor

Compararía casos representativos con una referencia manual, manteniendo las mismas reglas y registrando la asistencia humana. Mediría objetivos correctamente verificados, acciones inseguras evitadas, acciones válidas bloqueadas por error, duplicados y tiempo de coordinación frente a espera. Son métricas propuestas.

Un revisor independiente debería identificar petición, autoridad, cambio y resultado en el registro. Eso permite evaluar distintas soluciones válidas sin premiar una secuencia de herramientas prefijada.

| Material | Alcance |
|---|---|
| Diseño, documentación y recorridos | Presentados en este portfolio |
| Fragmentos recuperados del proyecto original | Contexto de revisión; no se publican como implementación completa |
| Runtime integrado equivalente a este caso | No demostrado por este paquete documental |

Orbit hace visible cómo abordo una automatización desde la responsabilidad operativa: defino el resultado, conecto la información necesaria, anticipo fallos y establezco qué evidencia permite confiar en el cierre.
