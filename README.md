# TP Final — ETL Exportaciones del NEA

Pipeline ETL desarrollado en Python para obtener, transformar y validar datos de exportaciones de las provincias del Nordeste Argentino (NEA).

## Provincias y período

El pipeline trabaja con:

* Chaco
* Corrientes
* Formosa
* Misiones

Período analizado: **1993–2024**.

## Fuente de datos

Los datos se obtienen mediante la API de Series de Tiempo de **datos.gob.ar / INDEC**.

Se utilizan las series correspondientes a:

* **Dataset 357.1:** exportaciones por provincia y país de destino.
* **Dataset 350.1:** exportaciones por provincia y rubro.

Unidad de los valores: **millones de dólares FOB**.

## ¿Qué hace el pipeline?

El proceso está dividido en tres etapas:

### 1. Extract

Se consultan las series de la API para las cuatro provincias.

Para cada provincia se obtienen:

* exportaciones por destino;
* exportaciones totales de la provincia;
* exportaciones por rubro.

En total se realizan 8 descargas: destino y rubro para cada una de las cuatro provincias.

### 2. Transform

Los datos obtenidos se transforman desde el formato original de la API a un dataset analítico.

Durante esta etapa se:

* convierten las fechas en años;
* transforma la estructura de datos de ancho a largo;
* clasifican los destinos por región geoeconómica;
* calcula la participación de cada destino sobre el total provincial;
* calcula la variación interanual;
* asigna la década correspondiente;
* calcula el ranking de destinos dentro de cada provincia y año;
* identifica los tres principales destinos;
* determina el rubro principal de cada provincia y año;
* calcula la participación de los productos primarios;
* integran los datos de destinos y rubros.

El resultado final contiene **13 columnas**.

### 3. Load

Antes de guardar los resultados se realizan controles de calidad:

* cantidad mínima de filas;
* presencia de las 13 columnas esperadas;
* unicidad de la clave `(provincia, año, destino)`;
* valores dentro de rangos razonables;
* cobertura de la información de rubros y variación interanual.

Luego se generan tres archivos:

```text
data/processed/exportaciones_nea.csv
data/processed/resumen.json
logs/pipeline.log
```

## ¿Cómo ejecutar el pipeline?

Desde la raíz del proyecto:

```powershell
python src/main.py
```

Para ejecutar los tests:

```powershell
python tests/test_transform.py
```

Los tests verifican el comportamiento de las principales funciones de transformación.

## Resultados de la ejecución

La ejecución final del pipeline produjo:

* **1.408 filas**
* **13 columnas**
* período **1993–2024**
* **4 provincias**
* valor mínimo: **0,00 millones de USD FOB**
* valor máximo: **399,14 millones de USD FOB**
* promedio: **20,17 millones de USD FOB**

Los controles de calidad finalizaron correctamente:

* 1.408 filas, superando el mínimo esperado de 1.000;
* 13 columnas correctas;
* 1.408 claves únicas;
* ninguna fila fuera del rango definido;
* ninguna fila sin información de rubro.

Las filas sin variación interanual corresponden al primer año disponible de cada serie, por lo que se consideran esperables.

## Principales resultados encontrados

Considerando el acumulado de las exportaciones durante todo el período 1993–2024, el orden de las provincias fue:

Misiones: 11.764,61 millones de USD FOB
Chaco: 9.719,76 millones de USD FOB
Corrientes: 5.801,55 millones de USD FOB
Formosa: 1.112,11 millones de USD FOB

Los principales destinos acumulados también presentan diferencias entre provincias:

En Chaco, la categoría Resto fue el principal destino acumulado, con 4.093,99 millones de USD FOB, seguida por China con 1.816,06 millones.
En Corrientes, Resto ocupó el primer lugar con 1.745,87 millones, seguido por Brasil con 1.638,40 millones.
En Formosa, Resto fue nuevamente el principal destino, con 336,47 millones, seguido por Brasil con 221,86 millones.
En Misiones, Brasil fue el principal destino acumulado con 2.700,87 millones, seguido muy de cerca por Estados Unidos con 2.592,39 millones.

La mayor observación individual del dataset corresponde a Corrientes → Brasil en 2020, con 399,14 millones de USD FOB.

Estos resultados muestran diferencias en la composición de los destinos de exportación entre las provincias del NEA y permiten utilizar el dataset para analizar tanto la evolución temporal como la importancia relativa de cada destino.

## Tests

El proyecto cuenta con **19 tests**, todos ejecutados correctamente:

```text
Ran 19 tests
OK
```

Además de los tests provistos por el template, se agregaron dos casos propios:

* comportamiento de `ancho_a_largo()` cuando recibe una lista vacía;
* cálculo de una década adicional mediante `calcular_decada()`.

## Estructura principal

```text
tp-final-et1/
├── src/
│   ├── extract.py
│   ├── transform.py
│   ├── load.py
│   └── main.py
├── tests/
│   └── test_transform.py
├── data/
│   ├── raw/
│   └── processed/
├── logs/
├── docs/
├── config.py
└── README.md
```

## Fuente

Fuente de datos: **INDEC**, vía el portal de datos abiertos del Estado argentino (**datos.gob.ar**).

IDs de series verificados el 2026-08-02.
