# 07. Estadísticas de diversidad genética con Pixy

## Introducción

Una vez obtenido un conjunto de variantes de alta calidad mediante el proceso de alineamiento, llamado y filtrado de variantes, el siguiente paso consiste en cuantificar la diversidad genética presente dentro y entre poblaciones. Estas estimaciones permiten describir la variabilidad genética, evaluar procesos evolutivos como la deriva genética, el flujo génico o la selección natural, e identificar regiones del genoma con patrones inusuales de variación.

En este capítulo se utilizará **Pixy**, una herramienta desarrollada para calcular estadísticas clásicas de genética de poblaciones a partir de archivos VCF que contienen tanto sitios variantes como invariantes (*all-sites VCF*). A diferencia de otros programas, Pixy produce estimaciones insesgadas incluso en presencia de datos faltantes (*missing data*), evitando la sobreestimación de parámetros como la diversidad nucleotídica (π).

Durante este ejercicio se calcularán tres de las estadísticas más utilizadas en genética de poblaciones:

- **Diversidad nucleotídica (π)**, que mide la variación genética dentro de una población.
- **Divergencia absoluta (dXY)**, que cuantifica las diferencias promedio entre dos poblaciones.
- **Índice de diferenciación genética (FST)**, que evalúa el grado de diferenciación genética entre poblaciones.

Los análisis se realizarán mediante ventanas deslizantes a lo largo del genoma, permitiendo visualizar cómo varían estos parámetros entre diferentes regiones genómicas.

---

## Objetivos

Al finalizar este capítulo el estudiante será capaz de:

- Comprender la importancia de utilizar sitios invariantes para estimar correctamente la diversidad genética.
- Preparar los archivos de entrada requeridos por Pixy.
- Definir poblaciones mediante un archivo de asignación de muestras.
- Calcular diversidad nucleotídica (π), divergencia genética (dXY) y diferenciación genética (FST).
- Interpretar los resultados obtenidos a lo largo del genoma.

---

# ¿Por qué utilizar Pixy?

Muchas herramientas calculan estadísticas de diversidad únicamente utilizando los SNPs presentes en un archivo VCF. Sin embargo, este enfoque introduce un sesgo importante, ya que ignora todas las posiciones del genoma donde no existe variación.

Por ejemplo, si un fragmento genómico posee 10 000 posiciones y solamente 120 corresponden a SNPs, calcular la diversidad utilizando únicamente esas 120 posiciones produce estimaciones artificialmente elevadas.

Pixy evita este problema utilizando archivos VCF que contienen **todos los sitios del genoma**, incluyendo tanto posiciones variables como invariantes, permitiendo obtener estimaciones más precisas de la diversidad genética.

---

# Estadísticas calculadas

## Diversidad nucleotídica (π)

La diversidad nucleotídica representa el número promedio de diferencias por sitio entre dos secuencias elegidas aleatoriamente dentro de una población.

Valores elevados indican una mayor variabilidad genética, mientras que valores bajos pueden reflejar eventos de selección, cuellos de botella poblacionales o tamaños efectivos reducidos.

---

## Divergencia genética (dXY)

La estadística dXY corresponde al número promedio de diferencias nucleotídicas por sitio entre individuos pertenecientes a dos poblaciones distintas.

Esta medida representa una estimación absoluta de la divergencia genética y resulta especialmente útil para comparar poblaciones con diferentes niveles de diversidad interna.

---

## Diferenciación genética (FST)

FST cuantifica la proporción de variación genética atribuible a diferencias entre poblaciones.

En términos generales:

- Valores cercanos a **0** indican poblaciones genéticamente similares.
- Valores intermedios sugieren diferenciación moderada.
- Valores altos reflejan una marcada estructura genética y un flujo génico limitado.

---

# Archivos necesarios

Para ejecutar Pixy se requieren los siguientes archivos:

- Archivo VCF comprimido (`.vcf.gz`) que incluya sitios variantes e invariantes.
- Índice del archivo VCF (`.tbi` o `.csi`).
- Archivo FASTA del genoma de referencia.
- Índice del genoma (`.fai`).
- Archivo con la asignación de individuos a poblaciones.

Ejemplo del archivo de poblaciones:

```text
sample    pop
IND01     Norte
IND02     Norte
IND03     Sur
IND04     Sur
```

---

# Flujo de trabajo

El análisis mediante Pixy sigue el siguiente flujo general:

1. Obtención del archivo VCF con todos los sitios del genoma.
2. Compresión e indexación del VCF.
3. Creación del archivo de poblaciones.
4. Ejecución de Pixy para calcular π, dXY y FST.
5. Visualización de los resultados en R mediante ventanas deslizantes.

---

# Salidas generadas

Pixy produce un archivo independiente para cada estadística solicitada.

Cada archivo contiene información como:

- Cromosoma.
- Posición inicial y final de la ventana.
- Número de sitios analizados.
- Número de sitios válidos.
- Valor estimado de la estadística.

Estos resultados pueden utilizarse posteriormente para construir gráficos que permitan identificar regiones con alta diversidad genética, zonas de elevada diferenciación entre poblaciones o posibles señales de selección natural.

---

# Ventajas de Pixy

- Utiliza sitios variantes e invariantes.
- Corrige el efecto de los datos faltantes.
- Produce estimaciones insesgadas de π, dXY y FST.
- Es compatible con archivos VCF generados mediante bcftools y GATK.
- Permite realizar análisis por ventanas deslizantes a lo largo de genomas completos.

---

## En este capítulo aprenderás

Al finalizar este ejercicio habrás construido un flujo de trabajo completo para estimar diversidad genética utilizando datos de secuenciación masiva. Los resultados obtenidos servirán como base para estudios de genética de poblaciones, genómica evolutiva, conservación, identificación de regiones bajo selección y comparación genética entre poblaciones o especies.
