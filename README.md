<p align="right">
<i>Erandeni Mendívil Morales</i><br>
<i>Universidad de Sonora</i><br>
<i>Octubre de 2026</i>
</p>

# Liga MX en XML: resultados y estadísticas validados con DTD

## Propósito

Diseñar un formato XML para representar los resultados y las estadísticas de los partidos de la Liga MX, y un DTD que defina las reglas que debe cumplir cualquier documento escrito en ese formato. Como caso de prueba se capturan los partidos del **domingo 27 de septiembre de 2026**, que corresponden a la jornada 10 del Torneo Apertura 2026.

## Estructura del repositorio

```text
liga-mx-xml/
├── README.md                    ← documentación del diseño (este archivo)
├── xml/
│   ├── resultados.xml           ← partidos del domingo 27/09/2026 (válido)
│   └── resultados-invalido.xml  ← pruebas negativas (inválido a propósito)
└── dtd/
    └── resultados.dtd           ← reglas del formato
```

## Fuente de los datos

[ESPN México: resultados de la Liga MX del 27 de septiembre de 2026](https://www.espn.com.mx/futbol/resultados/_/liga/mex.1/fecha/20260927), consultada el 6 de octubre de 2026. Las horas están en tiempo del centro de México (UTC−06:00).

## 1. Análisis de la información (Actividad 1)

Datos que ESPN publica para cada partido, agrupados como en la guía:

| Jornada | Partido | Estadísticas (por equipo) |
|---|---|---|
| Número de jornada | Equipo local y equipo visitante | Posesión |
| Competencia (Liga MX) | Marcador | Tiros |
| Torneo y temporada | Fecha y hora | Tiros a puerta |
| | Estadio y ciudad | Faltas |
| | Estado (finalizado, programado…) | Tiros de esquina |
| | Goleadores (minuto y tipo de gol) | Tarjetas (jugador, color y minuto) |
| | | Fueras de lugar y atajadas |

### Respuestas a las preguntas de análisis

1. **¿Cuál debería ser el elemento raíz?** `liga`. Agrupa todo el documento y permite que crezca a varias jornadas sin cambiar la raíz. La competencia, el torneo y la temporada se guardan una sola vez, como atributos de `liga`.
2. **¿Una jornada puede contener varios partidos?** Sí. La jornada 10 tuvo nueve partidos repartidos del viernes 25 al domingo 27; este documento contiene los tres del domingo. Por eso la fecha pertenece a cada partido y no a la jornada.
3. **¿Cada partido debe contener exactamente dos equipos?** Sí: uno local y uno visitante, ni más ni menos.
4. **¿Cómo distinguirían al equipo local del visitante?** Con dos elementos distintos, `equipoLocal` y `equipoVisitante`. Si se usara `<equipo condicion="local">` dos veces, el DTD solo podría exigir "dos elementos `equipo`" y no podría impedir dos locales; con nombres distintos sí puede exigir exactamente uno de cada uno.
5. **¿El marcador debe ser un solo dato o separar los goles?** Separar los goles: `golesLocal` y `golesVisitante`. Un texto como `2-3` obliga a partir la cadena para saber quién ganó o para calcular la diferencia de goles.
6. **¿Las estadísticas pertenecen al partido o a cada equipo?** A cada equipo: cada valor existe una vez por lado (posesión 54.3 contra 45.7). Se anidan dentro de `equipoLocal` y de `equipoVisitante` con la misma estructura.
7. **¿Qué datos son obligatorios?** El identificador y el estado del partido, la fecha, la hora, el estadio, los dos equipos (nombre y abreviatura) y, cuando hay estadísticas, las seis base de la guía: posesión, tiros, tiros a puerta, faltas, tiros de esquina y tarjetas.
8. **¿Cuáles podrían ser opcionales?** El marcador (un partido programado o pospuesto todavía no lo tiene), el bloque completo de estadísticas, los fueras de lugar y las atajadas, el detalle de goleadores y el tiempo agregado de un gol o de una tarjeta.

## 2. Modelo conceptual (Actividad 2)

El esquema de la guía se amplió con los datos generales del partido, las estadísticas dentro de cada equipo y el detalle de goles dentro del marcador. Los atributos aparecen con `@`:

```text
liga                         @nombre @torneo @temporada
└── jornada (1..n)           @numero
    └── partido (1..n)       @id @estado
        ├── fecha
        ├── hora             @zona
        ├── estadio          @ciudad
        ├── equipoLocal      @abreviatura
        │   ├── nombre
        │   └── estadisticas (0..1)
        │       ├── posesion
        │       ├── tiros
        │       ├── tirosAPuerta
        │       ├── faltas
        │       ├── tirosDeEsquina
        │       ├── fuerasDeLugar (0..1)
        │       ├── atajadas      (0..1)
        │       └── tarjetas
        │           └── tarjeta (0..n)   @color @minuto @agregado
        ├── equipoVisitante  @abreviatura   (misma estructura que equipoLocal)
        └── marcador (0..1)
            ├── golesLocal
            ├── golesVisitante
            └── gol (0..n)               @equipo @minuto @agregado @tipo
```

### Elemento o atributo

Criterio general: **lo que se muestra o se mide va en elementos; lo que identifica o clasifica a un elemento va en atributos.** Además, todo lo que se repite o tiene partes internas tiene que ser elemento (un atributo no puede repetirse ni tener hijos), y los valores que salen de una lista cerrada van en atributos enumerados, porque así el DTD puede restringirlos.

| Información | Elemento/Atributo | Justificación |
|---|---|---|
| Jornada | Elemento `jornada` con atributo `numero` | La jornada agrupa partidos (es un contenedor); su número solo la identifica. |
| Fecha | Elemento `fecha` | Es un dato del partido; cada partido de la jornada tiene la suya. |
| ID del partido | Atributo `id` (tipo `ID`) | Es un identificador interno, no contenido; como `ID`, el parser verifica que no se repita. |
| Equipo local | Elemento `equipoLocal` | Tiene estructura propia: nombre y estadísticas. |
| Equipo visitante | Elemento `equipoVisitante` | Igual que el local; un nombre distinto permite exigir exactamente uno de cada uno. |
| Goles | Elementos `golesLocal`, `golesVisitante` y `gol` | Son el dato central del resultado. Cada `gol` tiene contenido (el anotador) y atributos que lo describen (minuto, tipo). |
| Estadio | Elemento `estadio` con atributo `ciudad` | El nombre del estadio es contenido; la ciudad lo ubica. |
| Estado del partido | Atributo `estado` | Toma un valor de una lista cerrada (programado, en_juego, finalizado…) que el DTD puede validar. |
| Posesión | Elemento `posesion` | Es una medición del equipo; como elemento, agregar una estadística nueva no afecta a las demás. |
| Tarjetas | Elemento `tarjetas` con hijos `tarjeta` | Puede haber varias por equipo y un atributo no se puede repetir; el color y el minuto van como atributos de cada `tarjeta`. |
