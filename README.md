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

## 3. XML mínimo (Actividad 3)

`xml/resultados.xml` empezó con un solo partido, Pumas UNAM 2–3 Atlético de San Luis: identificador, estado, fecha, hora, estadio, los dos equipos y el marcador. Lista de comprobación de XML bien formado:

- [x] Un solo elemento raíz (`liga`).
- [x] Todas las etiquetas están cerradas.
- [x] El anidamiento es correcto y no hay etiquetas cruzadas.
- [x] Todos los valores de atributos van entre comillas.
- [x] La declaración `<?xml version="1.0" encoding="UTF-8"?>` está en la primera línea.

Comprobación automática (si no imprime nada, el documento está bien formado):

```bash
xmllint --noout xml/resultados.xml
```

## 4. Estadísticas (Actividad 4)

### Cómo relacionar cada estadística con su equipo

Se compararon dos diseños:

| | A. Estadísticas a nivel partido | B. Estadísticas dentro de cada equipo (elegido) |
|---|---|---|
| Ejemplo | `<posesion local="54.3" visitante="45.7"/>` | `<equipoLocal>` → `<estadisticas>` → `<posesion>54.3</posesion>` |
| Comprensión | Cada estadística se lee en pares | Cada equipo tiene todos sus datos juntos |
| Duplicación | Cada estadística repite el par local/visitante | Una sola declaración de `estadisticas` sirve para los dos equipos |
| Procesamiento | El mismo dato vive en dos atributos distintos | El mismo código o XPath sirve para ambos (`equipoLocal/estadisticas/tiros`) |
| Agregar una estadística | Un elemento nuevo con dos atributos | Una línea más en el modelo de `estadisticas` |

**Decisión:** diseño B, el mismo que muestra la guía (Partido → Local/Visitante → Estadísticas). Las tarjetas se registran una por una (jugador, color y minuto) en lugar de guardar un conteo: el número de amarillas se obtiene contándolas, así no existen dos datos que puedan contradecirse.

### Estadísticas seleccionadas

| Elemento | Nombre en ESPN | Valor | Cardinalidad |
|---|---|---|---|
| `posesion` | Posesión | Porcentaje con un decimal | Obligatoria |
| `tiros` | Tiros | Entero | Obligatoria |
| `tirosAPuerta` | A gol | Entero | Obligatoria |
| `faltas` | Faltas | Entero | Obligatoria |
| `tirosDeEsquina` | Tiros de esquina | Entero | Obligatoria |
| `fuerasDeLugar` | Fuera de lugar | Entero | Opcional |
| `atajadas` | Salvadas / atajadas | Entero | Opcional |
| `tarjetas/tarjeta` | Tarjetas amarillas y rojas | Una por tarjeta: jugador, color y minuto | Cero o más |

Las seis obligatorias son las de la guía. Fueras de lugar y atajadas son opcionales para que un partido con cobertura básica siga siendo válido.

En este paso también se agregó el detalle de goles dentro de `marcador`: cada `gol` indica a qué equipo se le cuenta, el minuto y el tipo (de jugada o de penal).

## 5. DTD: elementos y cardinalidades (Actividad 5)

`dtd/resultados.dtd` declara cada elemento con `<!ELEMENT …>`. Cardinalidades del modelo:

| Regla | Expresión DTD |
|---|---|
| Una liga contiene una o más jornadas | `<!ELEMENT liga (jornada+)>` |
| Una jornada contiene uno o más partidos | `<!ELEMENT jornada (partido+)>` |
| Un partido tiene exactamente un local | `equipoLocal`, sin operador, en `<!ELEMENT partido (fecha, hora, estadio, equipoLocal, equipoVisitante, marcador?)>` |
| Un partido tiene exactamente un visitante | `equipoVisitante`, sin operador, en la misma secuencia |
| Una estadística opcional | `fuerasDeLugar?` y `atajadas?` en `<!ELEMENT estadisticas (posesion, tiros, tirosAPuerta, faltas, tirosDeEsquina, fuerasDeLugar?, atajadas?, tarjetas)>` |
| Puede haber cero o más tarjetas | `<!ELEMENT tarjetas (tarjeta*)>` |

Otras decisiones:

- La coma (`,`) impone el orden de los hijos: un partido con el visitante antes que el local es inválido.
- `marcador?` es opcional porque un partido programado o pospuesto todavía no tiene resultado.
- `tarjetas` siempre está presente cuando hay estadísticas, aunque esté vacío: `<tarjetas/>` afirma "cero tarjetas", que no es lo mismo que "no hay datos".
- `<!ELEMENT marcador (golesLocal, golesVisitante, gol*)>`: un 0–0 tiene marcador pero ningún `gol`.
- Los nombres de los elementos van sin acentos (`posesion`, `estadisticas`) para evitar problemas de codificación entre herramientas.

## 6. Atributos (Actividad 6)

| Elemento | Atributo | Tipo | Uso o valor por omisión |
|---|---|---|---|
| `liga` | `nombre`, `torneo`, `temporada` | `CDATA` | `#REQUIRED` |
| `jornada` | `numero` | `CDATA` | `#REQUIRED` |
| `partido` | `id` | `ID` | `#REQUIRED` |
| `partido` | `estado` | `(programado \| en_juego \| finalizado \| suspendido \| pospuesto)` | `#REQUIRED` |
| `hora` | `zona` | `CDATA` | `"UTC-06:00"` |
| `estadio` | `ciudad` | `CDATA` | `#REQUIRED` |
| `equipoLocal`, `equipoVisitante` | `abreviatura` | `CDATA` | `#REQUIRED` |
| `tarjeta` | `color` | `(amarilla \| roja)` | `#REQUIRED` |
| `tarjeta`, `gol` | `minuto` | `NMTOKEN` | `#REQUIRED` |
| `tarjeta`, `gol` | `agregado` | `NMTOKEN` | `#IMPLIED` |
| `gol` | `equipo` | `(local \| visitante)` | `#REQUIRED` |
| `gol` | `tipo` | `(jugada \| penal)` | `"jugada"` |

- **`#REQUIRED`** para lo que identifica o clasifica sin excepción: el id, el estado, el color de una tarjeta.
- **`#IMPLIED`** para `agregado`: solo existe cuando el evento ocurre en tiempo de reposición.
- **Valor por omisión** cuando hay un caso normal: casi todos los goles son de jugada, así que `tipo` solo se escribe en los penales. La zona horaria es la del centro de México salvo que se indique otra; no se declaró `#FIXED` porque un estadio como el de Tijuana está en otra zona.
- **`NMTOKEN`** en `minuto` y `agregado`: no admite espacios ni símbolos como `'` o `+`. Por eso el tiempo de reposición se guarda en dos atributos (`minuto="45" agregado="1"`) en lugar de `minuto="45+1"`, lo que además permite ordenar los eventos numéricamente.

### ¿Qué ventaja tiene declarar el identificador como `ID` en lugar de `CDATA`?

Con `ID`, el parser validante:

1. **Garantiza que sea único en todo el documento.** Dos partidos `P001` producen un error de validación; con `CDATA` pasarían sin aviso.
2. **Exige que sea un nombre XML válido.** Debe empezar con una letra o un guion bajo: `P001` es válido, `001` no. Por eso los identificadores llevan el prefijo `P`.
3. **Lo vuelve referenciable.** Otros elementos pueden apuntarle con atributos `IDREF` (por ejemplo, una futura crónica `<resumen partido="P002">`, y el DTD verificaría que ese partido exista), y herramientas como `id('P002')` en XPath o `getElementById` en DOM lo encuentran directamente.

## 7. Asociación XML–DTD (Actividad 7)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE liga SYSTEM "../dtd/resultados.dtd">
```

- `liga` debe coincidir con el elemento raíz del documento.
- `SYSTEM` indica que el DTD es externo, en un archivo propio.
- La ruta es relativa **al archivo XML**, no a la carpeta desde donde se ejecuta el comando: desde `xml/` se sube un nivel (`..`) hasta `liga-mx-xml/` y se entra a `dtd/`. Por eso la validación funciona igual aunque se lance desde otra carpeta.

## Cómo validar

Con `xmllint` (viene en Linux y macOS; en Windows se puede usar desde WSL), ubicándose en la carpeta `liga-mx-xml/`:

```bash
# ¿Está bien formado? (solo revisa la sintaxis)
xmllint --noout xml/resultados.xml

# ¿Es válido? (sintaxis + reglas del DTD declarado en el DOCTYPE)
xmllint --noout --valid xml/resultados.xml
```

`--noout` evita que se imprima el documento y `--valid` activa la validación contra el DTD. **Si el comando no imprime nada (código de salida 0), el documento es válido**; si no, xmllint muestra cada error con su número de línea. Para validar contra un DTD sin depender del `DOCTYPE`: `xmllint --noout --dtdvalid dtd/resultados.dtd xml/resultados.xml`.

Alternativas:

- **IntelliJ IDEA** y **VS Code** (con la extensión XML de Red Hat) leen el `DOCTYPE`, validan contra el DTD y subrayan los errores en el editor.
- **Python con `lxml`**:

  ```bash
  python -c "from lxml import etree; etree.parse('xml/resultados.xml', etree.XMLParser(dtd_validation=True)); print('válido')"
  ```
