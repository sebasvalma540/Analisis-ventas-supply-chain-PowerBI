# 📊 Análisis de Ventas y Distribución Comercial en Supply Chain con Power BI — Proceso ETL y Modelo Dimensional


Proyecto de Business Intelligence end-to-end: extracción y transformación de un dataset público de cadena de suministro, construcción de un modelo dimensional en estrella y publicación de un reporte de tres páginas en Power BI con navegación, panel de filtros y medidas DAX con comparativo interanual.

## Autor

**Sebastian Valentin** — Estudiante de Ingeniería Industrial, UPC
GitHub: [sebasvalma540](https://github.com/sebasvalma540)

## 📩 Si quieres contactarme aqui te dejo mi linkedin =)
<p align="center">
  <a href="https://www.linkedin.com/in/sebastian-valentin-malpartida-11664428a/">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white" />
  </a>
</p> 


**Página 1 — General: Resumen Comercial**
![alt text](<Imagenes del dashboard/Pantalla general.png>)

**Página 2 — Seguimiento: Evaluación Mensual de Ventas**
![alt text](<Imagenes del dashboard/Pantalla seguimiento.png>)

**Página 3 — Mapa: Cobertura Geográfica de Clientes**
![alt text](<Imagenes del dashboard/Pantalla Mapa.png>)
---

## 1. Caso de negocio

**DataCo Global** es una empresa de comercio y distribución global que vende productos de consumo (ropa, deportes, electrónica, artículos para el hogar y mascotas) a clientes de distintos mercados y países, operando con varios modos de envío.

El problema que resuelve el tablero es que la operación genera cientos de miles de registros transaccionales de pedidos, envíos y entregas que viven en archivos planos: útiles para auditar, inservibles para decidir. Las áreas comercial y logística no tienen una vista única que responda preguntas básicas de gestión sin pedir una extracción manual cada vez.

### Objetivo general

Centralizar la información transaccional de pedidos y envíos de DataCo en un único modelo analítico y entregar a la gerencia comercial un tablero que permita monitorear el desempeño de ventas, rentabilidad y cobertura geográfica, con comparación contra el año anterior (LY) y capacidad de desagregar por mercado, departamento, categoría y modo de envío.

### Objetivos específicos

1. Consolidar y estandarizar los archivos fuente mediante un proceso ETL reproducible en Power Query.
2. Diseñar un modelo dimensional (esquema estrella) que separe hechos de dimensiones y habilite inteligencia de tiempo.
3. Definir los KPI comerciales del negocio: Venta Neta, Utilidad, Margen % y Ticket Promedio, con su variación interanual.
4. Entregar la lectura en tres niveles: resumen ejecutivo, evolución temporal y cobertura geográfica de la demanda.

### Preguntas de negocio que responde

- ¿Cuánto vendimos y cuánto ganamos en el período, y cómo nos comparamos contra el mismo período del año anterior?
- ¿Qué departamentos y categorías sostienen el margen y cuáles lo destruyen?
- ¿Cómo se comporta la venta mes a mes frente al año anterior y qué pasa con el volumen de unidades?
- ¿Qué segmento de cliente y qué medio de pago concentran la facturación?
- ¿Dónde están geográficamente nuestros clientes y qué países concentran la demanda?

---

## 2. Origen de los datos

| Ítem | Detalle |
|---|---|
| Dataset | *DataCo Smart Supply Chain for Big Data Analysis* |
| Plataforma de descarga | Kaggle — `shashwatwork/dataco-smart-supply-chain-for-big-data-analysis` |
| Fuente original | Mendeley Data, DOI [10.17632/8gx2fvg2k6.5](https://data.mendeley.com/datasets/8gx2fvg2k6/5) |
| Autores | Fabian Constante, Fernando Silva y António Pereira (Universidad Central del Ecuador / Instituto Politécnico de Leiria), publicado el 12 de marzo de 2019 |
| Archivo estructurado | `DataCoSupplyChainDataset.csv` |
| Diccionario de variables | `DescriptionDataCoSupplyChain.csv` |
| Archivo no estructurado (no utilizado) | `tokenized_access_logs.csv` (clickstream) |
| Período cubierto | Pedidos de 2015 a comienzos de 2018 |

El dataset registra las actividades de aprovisionamiento, producción, ventas y distribución comercial de la empresa. Cada fila corresponde a una línea de pedido e incluye datos del cliente, del producto, del destino, montos de venta y utilidad, y los tiempos de envío programados y reales.

> **Nota sobre el alcance:** se trabajó únicamente con el archivo estructurado. El archivo de clickstream quedó fuera del alcance por no aportar al caso de negocio comercial planteado.

---

## 3. Proceso ETL (Power Query)

La carga no apunta a un archivo suelto sino a una **carpeta**, de modo que el modelo pueda refrescarse agregando nuevos archivos sin reconfigurar el origen.

### 3.1 Extracción

El árbol de dependencias de consulta sigue el patrón estándar de *Combinar archivos* de Power Query:

![alt text](<Imagenes del dashboard/Proceso ETL - Modelado.png>)

### 3.2 Transformación

Sobre `Data` se aplicaron, entre otras:

- Promoción de encabezados y tipificación explícita de columnas (fechas, enteros, decimales, texto).
- Renombrado de campos al español para que la capa semántica sea legible por el usuario de negocio.
- Eliminación de columnas no utilizadas en el caso (campos redundantes y de identificación interna).
- Normalización de campos de fecha de pedido y fecha de envío.
- Depuración de valores nulos y filas inconsistentes.

### 3.3 Carga — generación de las tablas del modelo

Desde `Data` se derivan por referencia las dimensiones y la tabla de hechos. Cada dimensión se construye seleccionando sus columnas, **eliminando duplicados** sobre la clave de negocio y validando unicidad:

| Consulta | Tipo | Se obtiene de |
|---|---|---|
| `Dim_Cliente` | Dimensión | Referencia a `Data` |
| `Dim_Producto` | Dimensión | Referencia a `Data` |
| `Dim_Categoria` | Dimensión | Referencia a `Data` |
| `Dim_Ubicación` | Dimensión | Referencia a `Data` |
| `Dim_Envio` | Dimensión | Referencia a `Data` |
| `Fact_Ventas_Envios` | Hechos | Referencia a `Data` |
| `Dim_Calendario` | Dimensión de tiempo | Generada a partir del rango de fechas de `Fact_Ventas_Envios` |

El calendario se construye dinámicamente tomando la fecha mínima y máxima de la tabla de hechos, de modo que crece solo si crece la data.

---

## 4. Modelo de datos

Esquema en **estrella**, con una ramificación tipo copo de nieve entre producto y categoría.

![alt text](<Imagenes del dashboard/Modelado de datos.png>)


### 4.1 Tabla de hechos — `Fact_Ventas_Envios`

Granularidad: una fila por línea de pedido.

| Campo | Tipo | Rol |
|---|---|---|
| `Venta neta` | Numérico | Métrica — base de `Suma Venta neta` |
| `Utilidad` | Numérico | Métrica — base de `Suma Utilidad` |
| `Cantidad` | Numérico | Métrica — unidades vendidas |
| `Días de envío programados` | Numérico | Métrica logística — compromiso de entrega |
| `Días de envío reales` | Numérico | Métrica logística — ejecución real |
| `Entrega tardía` | Numérico | Indicador binario (1 = tardía, 0 = a tiempo) |
| `Fecha de pedido` | Fecha | Clave de tiempo — **relación activa** con `Dim_Calendario` |
| `Fecha de envío` | Fecha | Fecha de evento, sin relación al calendario |
| `ID de pedido por cliente` | Numérico | Clave degenerada del pedido — denominador de `Ticket Promedio` |
| `ID de pedido por producto` | Numérico | Clave degenerada de la línea de pedido |
| `Tipo de pago` | Texto | **Dimensión degenerada** — alimenta el gráfico de venta por medio de pago |
| `ID de cliente` | Numérico | FK → `Dim_Cliente` |
| `ID de producto` | Numérico | FK → `Dim_Producto` |
| `ID de ubicación` | Numérico | FK → `Dim_Ubicación` |
| `ID de envío` | Numérico | FK → `Dim_Envio` |

**Nota sobre las claves:** `ID de pedido por cliente`, `ID de pedido por producto` y las claves foráneas quedaron tipificadas como numéricas, por lo que Power BI las marca con el símbolo de sumatoria (Σ) y las agrega por defecto. Conviene ponerles *Resumen predeterminado = No resumir* para evitar que un arrastre accidental al lienzo produzca la suma de identificadores, que no tiene significado de negocio.

`Tipo de pago` es el único atributo descriptivo que vive dentro de la tabla de hechos. Se trata de una dimensión degenerada: no justifica una tabla propia porque tiene pocas categorías y no comparte atributos con nada más.

### 4.2 Dimensiones

| Tabla | Atributos | Cardinalidad |
|---|---|---|
| `Dim_Cliente` | ID de cliente, Segmento de cliente, Latitud, Longitud | 1 : * |
| `Dim_Ubicación` | ID de ubicación, País de destino, Región de destino, Estado/Provincia de destino, Ciudad de destino, Mercado | 1 : * |
| `Dim_Producto` | ID de producto, Producto, ID de categoría | 1 : * |
| `Dim_Categoria` | ID de categoría, Categoría, Departamento | 1 : * hacia `Dim_Producto` |
| `Dim_Envio` | ID de envío, Modo de envío, Estatus de entrega | 1 : * |
| `Dim_Calendario` | Fecha, Año, Trimestre, AñoTrimestre, Mes, NúmeroMes, AñoMes, NúmeroSemana, DíaSemana, NúmeroDíaSemana, NúmeroDíasMes, EsFinDeSemana | 1 : * |

`Dim_Calendario` incluye tanto los campos de visualización (Mes, Trimestre, DíaSemana) como sus equivalentes numéricos (NúmeroMes, NúmeroSemana, NúmeroDíaSemana), que se usan para ordenar correctamente los textos mediante *Ordenar por columna*. Sin esto, los meses aparecerían en orden alfabético en el eje del gráfico de la página Seguimiento.

### 4.3 Relaciones

| Origen | Destino | Clave | Cardinalidad | Dirección del filtro |
|---|---|---|---|---|
| `Dim_Cliente` | `Fact_Ventas_Envios` | ID de cliente | 1 : * | Simple |
| `Dim_Ubicación` | `Fact_Ventas_Envios` | ID de ubicación | 1 : * | Simple |
| `Dim_Producto` | `Fact_Ventas_Envios` | ID de producto | 1 : * | Simple |
| `Dim_Envio` | `Fact_Ventas_Envios` | ID de envío | 1 : * | Simple |
| `Dim_Calendario` | `Fact_Ventas_Envios` | Fecha → Fecha de pedido | 1 : * | Simple |
| `Dim_Categoria` | `Dim_Producto` | ID de categoría | 1 : * | Simple |

Todas las relaciones son de **uno a muchos con filtro cruzado simple**, desde la dimensión hacia la tabla de hechos: el filtro propaga en una sola dirección y no hay ambigüedad de caminos. `Dim_Calendario` está marcada como tabla de fechas para habilitar las funciones de inteligencia de tiempo (`DATEADD`) que usan todas las medidas de variación interanual.

La única ramificación de copo de nieve es `Dim_Categoria → Dim_Producto`, que permite la jerarquía Departamento → Categoría de la matriz de la página General sin duplicar el departamento en cada fila de producto.

---

## 5. Medidas DAX

Las medidas están organizadas en **tablas de medidas vacías** (carpetas lógicas) para separar el cálculo del dato:

```
Medida_Fecha_Maxima
Medidas_Venta_Neta
Medidas_Utilidad
Medidas_Margen_%
Medidas_Ticket_Promedio
Subtitulo Dinamico
```

Cada grupo de KPI sigue el mismo patrón de cuatro capas:

1. **Medida base** — el cálculo numérico puro.
2. **Medida de formato dinámico** — convierte el número a `K / M / bn` según su magnitud.
3. **Variación numérica** — compara contra el año anterior con `DATEADD`.
4. **Variación en texto** — arma la etiqueta con flecha y signo que se muestra bajo cada tarjeta.

### 5.1 Venta Neta

```dax
Suma Venta neta = SUM(Fact_Ventas_Envios[Venta neta])
```

```dax
Suma Venta LY =
VAR ANO_ACTUAL = [Suma Venta neta]
VAR ANO_PASADO = CALCULATE([Suma Venta neta], DATEADD(Dim_Calendario[Fecha], -1, YEAR))
RETURN ANO_PASADO
```

```dax
Suma_VentNeta_Dinamico =
VAR Valor = [Suma Venta neta]
VAR AbsValor = ABS(Valor)
RETURN
    SWITCH(
        TRUE(),
        AbsValor >= 1000000000, FORMAT(Valor / 1000000000, "#,##0.0") & " bn",
        AbsValor >= 1000000,    FORMAT(Valor / 1000000, "#,##0.0") & " M",
        AbsValor >= 1000,       FORMAT(Valor / 1000, "#,##0.0") & " K",
        FORMAT(Valor, "#,##0")
    )
```

```dax
Variacion Venta Neta Numero =
VAR ANO_ACTUAL = [Suma Venta neta]
VAR ANO_PASADO = CALCULATE([Suma Venta neta], DATEADD(Dim_Calendario[Fecha], -1, YEAR))
VAR VARIACION = DIVIDE(ANO_ACTUAL - ANO_PASADO, ANO_PASADO, 0)
RETURN VARIACION
```

```dax
Variacion Venta Neta % Texto =
VAR MEDIDA = [Variacion Venta Neta Numero]
VAR REDONDEO = CONCATENATE(ROUND([Variacion Venta Neta Numero] * 100, 3), "%")
RETURN
    SWITCH(
        TRUE(),
        [Variacion Venta Neta Numero] < 0, "LY: " & "▼ " & REDONDEO,
        "LY: " & "▲ +" & REDONDEO
    )
```

`Suma Venta LY` se usa como serie independiente en el gráfico de columnas de la página Seguimiento, mientras que `Variacion Venta Neta % Texto` alimenta la etiqueta bajo la tarjeta KPI de la página General. Son dos consumos distintos del mismo cálculo interanual: uno devuelve el valor del año anterior y el otro, el crecimiento relativo ya formateado.

### 5.2 Utilidad

```dax
Suma Utilidad = SUM(Fact_Ventas_Envios[Utilidad])
```

```dax
Suma_Utilidad_Dinamico =
VAR Valor = [Suma Utilidad]
VAR AbsValor = ABS(Valor)
RETURN
    SWITCH(
        TRUE(),
        AbsValor >= 1000000000, FORMAT(Valor / 1000000000, "#,##0.0") & " bn",
        AbsValor >= 1000000,    FORMAT(Valor / 1000000, "#,##0.0") & " M",
        AbsValor >= 1000,       FORMAT(Valor / 1000, "#,##0.0") & " K",
        FORMAT(Valor, "#,##0")
    )
```

```dax
Variacion Utilidad Numero =
VAR ANO_ACTUAL = [Suma Utilidad]
VAR ANO_PASADO = CALCULATE([Suma Utilidad], DATEADD(Dim_Calendario[Fecha], -1, YEAR))
VAR VARIACION = DIVIDE(ANO_ACTUAL - ANO_PASADO, ANO_PASADO, 0)
RETURN VARIACION
```

```dax
Variacion Utilidad % Texto =
VAR MEDIDA = [Variacion Utilidad Numero]
VAR REDONDEO = CONCATENATE(ROUND([Variacion Utilidad Numero] * 100, 3), "%")
RETURN
    SWITCH(
        TRUE(),
        [Variacion Utilidad Numero] < 0, "LY: " & "▼ " & REDONDEO,
        "LY: " & "▲ +" & REDONDEO
    )
```

### 5.3 Margen %

```dax
Margen Numero =
VAR NUMERO = DIVIDE(Medidas_Utilidad[Suma Utilidad], Medidas_Venta_Neta[Suma Venta neta], 0)
RETURN NUMERO
```

```dax
Margen % =
VAR NUMERO = ROUND(DIVIDE(Medidas_Utilidad[Suma Utilidad], Medidas_Venta_Neta[Suma Venta neta], 0), 4) * 100
VAR NUMERO_PORCENTAJE = CONCATENATE(NUMERO, " %")
RETURN NUMERO_PORCENTAJE
```

```dax
Variacion Margen Numero =
VAR ANO_ACTUAL = [Margen Numero]
VAR ANO_PASADO = CALCULATE([Margen Numero], DATEADD(Dim_Calendario[Fecha], -1, YEAR))
VAR VARIACION = ANO_ACTUAL - ANO_PASADO
RETURN VARIACION
```

```dax
Variacion Margen % Texto =
VAR MEDIDA = [Variacion Margen Numero]
VAR REDONDEO = CONCATENATE(FORMAT([Variacion Margen Numero], "0.0000") * 100, " PT")
RETURN
    SWITCH(
        TRUE(),
        [Variacion Margen Numero] < 0, "LY: " & "▼ " & REDONDEO,
        "LY: " & "▲ +" & REDONDEO
    )
```

La variación del margen se expresa en **puntos porcentuales (PT)**, no en porcentaje, porque se trata de la diferencia entre dos ratios y no de un crecimiento relativo. Por eso esta medida usa una resta simple (`ANO_ACTUAL - ANO_PASADO`) mientras que las demás variaciones usan `DIVIDE` para obtener el crecimiento proporcional.

### 5.4 Ticket Promedio

```dax
Ticket Promedio =
VAR TOTAL_PEDIDOS = DISTINCTCOUNT(Fact_Ventas_Envios[ID de pedido por cliente])
VAR TICKET = ROUND(DIVIDE(Medidas_Venta_Neta[Suma Venta neta], TOTAL_PEDIDOS, 0), 1)
RETURN TICKET
```

```dax
Variacion Ticket Promedio Numero =
VAR ANO_ACTUAL = [Ticket Promedio]
VAR ANO_PASADO = CALCULATE([Ticket Promedio], DATEADD(Dim_Calendario[Fecha], -1, YEAR))
VAR VARIACION = DIVIDE(ANO_ACTUAL - ANO_PASADO, ANO_PASADO, 0)
RETURN VARIACION
```

El uso de `DISTINCTCOUNT` sobre el identificador de pedido evita inflar el denominador: un pedido con cinco líneas sigue siendo un solo ticket.

### 5.5 Encabezado dinámico

```dax
Subtitulo =
"Año: " & SELECTEDVALUE(Dim_Calendario[Año], "Todos") &
" | Mercado: " & SELECTEDVALUE('Dim_Ubicación'[Mercado], "Todos") &
" | Departamento: " & SELECTEDVALUE(Dim_Categoria[Departamento], "Todos") &
" | Modo de envío: " & SELECTEDVALUE('Dim_Envio'[Modo de envío], "Todos")
```

La medida `Subtitulo` se coloca en una tarjeta bajo el título de cada página. `SELECTEDVALUE` devuelve el valor del segmentador cuando hay una sola selección y el texto `"Todos"` cuando hay varias o ninguna, de modo que el encabezado siempre declara el contexto de filtro vigente. Esto hace que cualquier captura o exportación del reporte sea autoexplicativa: el lector sabe exactamente qué recorte de datos está viendo sin tener que abrir el panel de filtros.

```dax
Fecha_Actualizado_al =
VAR FECHA_MAXIMA = MAX(Dim_Calendario[Fecha])
RETURN "Actualizado al " & UNICHAR(10) & FORMAT(FECHA_MAXIMA, "DD/MM/YYYY")
```

`MAX` responde al contexto de filtro, por lo que la etiqueta devuelve la última fecha **visible** y no la última fecha del modelo: en la página General con el filtro Año = 2016 muestra `31/12/2016`, mientras que en las páginas sin filtro de año muestra `06/02/2018`, el cierre del dataset. `UNICHAR(10)` inserta un salto de línea para que el texto se divida en dos renglones dentro de la píldora turquesa del encabezado, sin necesidad de dos tarjetas superpuestas.

---

## 6. Diseño y configuración del reporte

### 6.1 Navegación

- Barra lateral fija con botones **General**, **Seguimiento** y **Mapa**, implementada con marcadores y acciones de navegación de página.
- Panel superior izquierdo con dos íconos: **abrir panel de filtros** y **cerrar panel**, construido con marcadores sobre un grupo de segmentadores (Año, Mercado, Departamento, Modo de envío) que se despliega sobre el lienzo.

### 6.2 Identidad visual

Paleta de tres colores (turquesa `#1BA8B8`, gris carbón para la barra de navegación y blanco/gris claro para el fondo del lienzo), tipografía única, tarjetas con esquinas redondeadas y sombra suave, y encabezado con logotipo de la empresa en todas las páginas.

### 6.3 Páginas

**Página 1 — General: Resumen Comercial**

| Elemento | Contenido |
|---|---|
| Tarjetas KPI | Venta Neta, Utilidad, Margen %, Ticket Promedio, cada una con su variación LY y flecha condicional |
| Matriz jerárquica | Departamento → Categoría, con Venta Neta, Utilidad, Margen % y Ticket Promedio |
| Gráfico de barras | Venta neta por `Fact_Ventas_Envios[Tipo de pago]` |
| Gráfico de anillo | Venta neta por `Dim_Cliente[Segmento de cliente]` |

Lectura del período 2016: Venta Neta 11.1 M, Utilidad 1.3 M, Margen 11.86 % y Ticket Promedio 529.80. Venta y utilidad crecen frente al año anterior, pero el margen cae 0.04 PT y el ticket promedio retrocede 0.132 %: el crecimiento viene de más pedidos, no de mejores pedidos.

**Página 2 — Seguimiento: Evaluación Mensual de Ventas**

| Elemento | Contenido |
|---|---|
| Columnas agrupadas | Venta neta vs Venta LY por mes |
| Gráfico de área | Cantidad total (unidades) por mes |

Permite detectar estacionalidad y quiebres de tendencia. La caída de volumen en el último trimestre corresponde al corte del dataset, no a un fenómeno comercial.

**Página 3 — Mapa: Cobertura Geográfica de Clientes**

| Elemento | Contenido |
|---|---|
| Mapa de burbujas | **Recuento** de `ID de pedido por cliente`, ubicado por `Dim_Cliente[Latitud]` y `[Longitud]` |
| Barras | **Recuento** de `ID de pedido por cliente` por `Dim_Ubicación[País de destino]` |

Ambos visuales usan recuento y no suma, de modo que la burbuja y la barra miden cantidad de pedidos y no el valor acumulado de un identificador. Esto mantiene coherencia entre las dos lecturas de la página: lo que se ve en el mapa es la misma métrica que ordena el ranking de países.

Muestra la concentración de la demanda: Estados Unidos encabeza con ~25 mil pedidos, seguido de Francia y México con ~13 mil cada uno.

### 6.4 Interactividad

- Todos los visuales responden a los segmentadores del panel lateral.
- Filtrado cruzado activo entre visuales de una misma página.
- Jerarquía expandible Departamento → Categoría en la matriz.
- Formato condicional en las etiquetas de variación: verde con ▲ cuando el indicador mejora, rojo con ▼ cuando empeora, resuelto en DAX con `SWITCH(TRUE(), …)`.

---

## 7. Cómo reproducir el proyecto

1. Descargar `DataCoSupplyChainDataset.csv` desde Kaggle o desde el repositorio Mendeley.
2. Colocarlo en una carpeta local dedicada (el origen del modelo es la carpeta, no el archivo).
3. Abrir el archivo `.pbix` y actualizar el **Parámetro1** con la ruta de esa carpeta.
4. Ejecutar *Actualizar* y verificar en la vista Modelo que las siete tablas estén cargadas y relacionadas.
5. Confirmar que `Dim_Calendario` siga marcada como tabla de fechas.

---

## 8. Consideraciones y mejoras pendientes

- **Formato de la variación de margen:** en `Variacion Margen % Texto`, la expresión `FORMAT([Variacion Margen Numero], "0.0000") * 100` multiplica por 100 el resultado de un `FORMAT`, que devuelve texto. DAX resuelve la operación con una conversión implícita de texto a número y el resultado es correcto, pero depende del separador decimal de la configuración regional. La versión robusta es `CONCATENATE(ROUND([Variacion Margen Numero] * 100, 4), " PT")`, que opera sobre el número antes de formatear, igual que hacen las otras tres medidas de variación.
- **Campos logísticos sin explotar:** `Días de envío programados`, `Días de envío reales` y `Entrega tardía` ya están en el modelo pero aún no tienen visuales asociados. Una cuarta página de cumplimiento de entregas (% de entregas tardías por modo de envío y por mercado) es la extensión natural del tablero.
- **Dimensión de tiempo del envío:** hoy solo `Fecha de pedido` tiene relación activa con el calendario. Para analizar el desempeño por fecha de despacho haría falta una relación inactiva sobre `Fecha de envío` activada con `USERELATIONSHIP`.
- **Naturaleza del dato:** el dataset es una fuente académica de referencia, no datos productivos de una empresa real, por lo que las conclusiones son válidas como ejercicio analítico y no como diagnóstico de negocio.

---

## 9. Referencias

- Constante, F., Silva, F. y Pereira, A. (2019). *DataCo SMART SUPPLY CHAIN FOR BIG DATA ANALYSIS* (Versión 5) [Conjunto de datos]. Mendeley Data. https://doi.org/10.17632/8gx2fvg2k6.5
- Kaggle. *DataCo Smart Supply Chain for Big Data Analysis*. https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis