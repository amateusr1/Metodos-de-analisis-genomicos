# NGS Data Analysis - Variant Calling

> Una guía para el procesamiento de BAMs y llamada de variantes y un flujo de trabajo completo con GATK
---
# Introducción

El objetivo del llamado de variantes consiste en identificar todas las posiciones del genoma donde las muestras presentan diferencias respecto al genoma de referencia, estas diferencias reciben el nombre de **variantes genéticas**. Las variantes más comunes son: **SNP**, cambio de un único nucleótido; **INDEL** inserción o deleción de uno o varios nucleótidos; **MNP**, sustitución simultánea de varios nucleótidos consecutivos; **Variantes estructurales** incluyen, inversiones, translocaciones, duplicaciones, grandes inserciones, grandes deleciones, estas ultimas generalmente requieren algoritmos especializados distintos de los llamadores convencionales de SNPs.

El llamado de variantes constituye un proceso de inferencia estadística cuyo objetivo es distinguir diferencias genéticas reales de los errores introducidos durante la obtención y procesamiento de los datos de secuenciación. Aunque las plataformas modernas de NGS presentan tasas de error relativamente bajas, ningún experimento de secuenciación está completamente libre de errores. En consecuencia, no toda discrepancia observada entre una lectura y el genoma de referencia corresponde necesariamente a una variante biológica.

Los primeros algoritmos de llamado de variantes utilizaban reglas simples basadas en el conteo de nucleótidos observados en cada posición. Sin embargo, este enfoque resultó insuficiente para analizar regiones con baja cobertura, errores de secuenciación o variantes complejas. Actualmente, prácticamente todos los llamadores modernos emplean modelos probabilísticos basados en inferencia bayesiana, estos modelos integran simultáneamente la información proveniente de: calidad de las bases; calidad del alineamiento; profundidad de cobertura; frecuencia esperada de variantes; distribución de alelos.

Adionalmente el llamado de variantes no solamente identifica la presencia de una mutación, también determina el **genotipo** del individuo, en organismos diploides existen tres posibilidades básicas:

| Genotipo | Significado |
|-----------|-------------|
| 0/0 | Homocigoto referencia |
| 0/1 | Heterocigoto |
| 1/1 | Homocigoto alternativo |

La determinación correcta del genotipo depende de algunos de estos mismos factores: profundidad de cobertura; calidad Phred; calidad del alineamiento; distribución de alelos; probabilidad estadística.

El resultado final del llamado de variantes corresponde al genotipo con mayor probabilidad posterior.

## Manejo de fuentes de error en el llamado de variantes 

Las principales fuentes de error durante el llamado de variantes pueden clasificarse en errores experimentales, errores computacionales y errores derivados del propio genoma de referencia. El reconocimiento y control de estos factores constituye uno de los aspectos más importantes del análisis bioinformático. 

1. Durante la síntesis y detección de nucleótidos pueden producirse errores en la identificación de las bases. Estos errores aumentan generalmente hacia los extremos de las lecturas debido a la disminución progresiva de la intensidad de fluorescencia y al incremento del ruido instrumental. Cada base recibe una puntuación de calidad Phred que representa la probabilidad de que dicha base haya sido identificada incorrectamente. Una baja calidad de secuenciación incrementa la probabilidad de detectar falsos SNPs o falsos indels. Estos errores se pueden controlar durante el trimming de lecturas de baja calidad con herramientas como Trimmomatic, fastp, o el altamente recomendado módulo clean de Captus (https://edgardomortiz.github.io/captus.docs/) y filtrando sitios con puntuación QUAL baja en el VCF final. Es posible que sea necesario cortar los extremos de las lecturas.

La escala Phred representa logarítmicamente la probabilidad de error durante la identificación de una base. Su definición matemática es:

```
Q=-10\log_{10}(P_{error})
```

donde:

- **Q** corresponde a la calidad Phred.
- **P_error** representa la probabilidad de error.

Por ejemplo:

| Calidad | Probabilidad de error |
|---------|-----------------------|
| Q20 | 1 % |
| Q30 | 0.1 % |
| Q40 | 0.01 % |

Debido a esta relación logarítmica, pequeñas diferencias en la puntuación Phred representan cambios importantes en la confiabilidad de las bases.

La **recalibración de la calidad de bases (Base Quality Score Recalibration, BQSR)** constituye una etapa de preprocesamiento que se realiza después del alineamiento, la ordenación del archivo BAM y el marcado de duplicados, pero antes del llamado de variantes con HaplotypeCaller. La herramienta BaseRecalibrator de GATK analiza las puntuaciones de calidad asignadas por el secuenciador y estima los errores sistemáticos asociados al ciclo de secuenciación, al contexto nucleotídico y a otras covariables. Para evitar confundir variantes biológicas con errores de secuenciación, el algoritmo utiliza un conjunto de variantes conocidas como referencia. Con esta información genera un modelo estadístico que posteriormente es aplicado mediante ApplyBQSR, el cual recalibra las puntuaciones Phred de cada base sin modificar la secuencia de nucleótidos. El archivo BAM resultante contiene estimaciones de calidad más precisas, lo que mejora la exactitud del ensamblaje local, el cálculo de probabilidades mediante PairHMM y, en consecuencia, el llamado de variantes realizado por HaplotypeCaller.

2. La amplificación mediante PCR puede introducir mutaciones artificiales que posteriormente son interpretadas como variantes reales. Además, la sobreamplificación puede originar múltiples copias idénticas del mismo fragmento de ADN, conocidas como duplicados de PCR. Estos duplicados generan una representación artificialmente elevada de determinados alelos y pueden sesgar la estimación del genotipo. En el módulo anterior explico como realizar el marcado de duplicados mediante Picard MarkDuplicates. 

3. Los errores de mapeo constituyen una de las principales causas de falsos positivos durante el llamado de variantes. Este problema ocurre principalmente cuando una lectura puede alinearse con alta similitud en múltiples regiones del genoma, situación frecuente en secuencias repetitivas, familias multigénicas o regiones altamente conservadas. Los alineadores asignan un valor de Mapping Quality (MAPQ) para estimar la probabilidad de que una lectura haya sido ubicada correctamente. Se filtran aplicando un umbral mínimo de MAPQ (típicamente MAPQ ≥ 20) durante el llamado de variantes, y excluyendo lecturas que mapean en múltiples posiciones del genoma (multimappers).

4. El genoma de referencia representa únicamente un individuo o ensamblaje específico de la especie y no necesariamente refleja toda la diversidad genética existente. Errores de ensamblaje, regiones faltantes, inversiones o secuencias incorrectamente ensambladas pueden provocar discrepancias sistemáticas entre las lecturas y la referencia. Los errores asociados a la referencia se mitigan usando genomas de alta calidad, y en poblaciones muy divergentes de la referencia se recomienda el uso de grafos de pangenoma (pangenome graphs) como alternativa al mapeo lineal.

5. La profundidad de secuenciación determina el número de lecturas que cubren una determinada posición del genoma. Una cobertura baja disminuye considerablemente la confianza en la estimación del genotipo debido a que un pequeño número de errores experimentales puede confundirse con variantes verdaderas. Por el contrario, coberturas excesivamente altas pueden indicar la presencia de duplicados de PCR o regiones repetitivas. Esto se puede controlar filtrando sitios por profundidad mínima (DP ≥ 10) y máxima (DP ≤ media + 2×SD) en el VCF. 

---
# Fundamento estadístico del llamado de variantes

El llamado de variantes es un problema de inferencia estadística: dado un conjunto de lecturas ¿cuál es el genotipo más probable para este individuo en esta posición? La respuesta se construye en tres pasos sucesivos.

**Paso 1 — Genotype Likelihood: ¿qué tan bien explica cada genotipo mis lecturas?**

El primer paso consiste en calcular, para cada genotipo posible, la probabilidad de observar exactamente las lecturas que se obtuvieron si ese genotipo fuera el verdadero. Esta probabilidad se denomina Genotype Likelihood. Cada lectura que cubre una posición aporta evidencia a favor o en contra de cada genotipo. El likelihood de todas las lecturas juntas se calcula multiplicando la contribución de cada lectura individual, incorporando su puntuación de calidad Phred (la probabilidad de que esa base esté mal identificada). Una lectura de alta calidad pesa más que una de baja calidad.

**Paso 2 — Inferencia bayesiana: incorporar conocimiento previo**

Solo con el Genotype Likelihood no es suficiente para definir el genotipo. En el caso de una posición con solo 2 lecturas, una A y una B. El likelihood favorecería al heterocigoto AB, pero con solo 2 lecturas no se puede descartar que sea un homocigoto AA con un error de secuenciación en una lectura.

La inferencia bayesiana constituye el fundamento estadístico de herramientas como GATK, FreeBayes y bcftools. El objetivo consiste en calcular la probabilidad del genotipo considerando toda la evidencia disponible. El teorema de Bayes combina el likelihood con un prior — un conocimiento previo sobre qué tan probable es cada genotipo antes de mirar las lecturas:

El genotipo seleccionado es aquel con la **mayor probabilidad posterior (MAP)**. Los priors representan el conocimiento biológico disponible antes del análisis. Dependiendo del algoritmo y del contexto pueden construirse a partir de:

- Tasa esperada de heterocigosidad de la especie (p. ej., en humanos θ ≈ 0.001)
- Frecuencias alélicas poblacionales de bases de datos como dbSNP o gnomAD
- Equilibrio de Hardy-Weinberg: si se conoce la frecuencia del alelo en la población, se puede calcular la probabilidad esperada de cada genotipo
- Información de múltiples individuos analizados conjuntamente (genotipado conjunto)

En regiones de baja cobertura, el likelihood tiene mucha incertidumbre. Un prior bien calibrado actúa como un "ancla" que evita llamar variantes falsas. Por ejemplo, si la tasa de heterocigosidad esperada es 0.001 (1 en 1,000 posiciones), el prior para una variante es bajo. Con solo 2 lecturas discordantes, la probabilidad posterior de una variante real sigue siendo baja aunque el likelihood la favorezca. Con 20 lecturas discordantes, la evidencia del likelihood supera al prior y se llama la variante.

**Paso 3 — Genotipado conjunto**

Cuando varias muestras son analizadas simultáneamente, la estimación de los genotipos se fortalece incorporando información poblacional. El supuesto más utilizado es el equilibrio de Hardy-Weinberg (HWE). Si la frecuencia del alelo A en la población es p y la del alelo B es q = 1 - p, las frecuencias esperadas de los genotipos son:

```
P(AA)=p2    P(AB)=2pq    P(BB)=q2
```

Estas frecuencias genotípicas se usan como priors en el modelo bayesiano conjunto. La clave es que las frecuencias alélicas se estiman a partir de todas las muestras simultáneamente, lo que tiene dos ventajas importantes:

1. Mayor sensibilidad para variantes raras: Si un alelo B aparece en solo 1 de 20 individuos, su frecuencia estimada es baja pero no cero. Esto permite detectar variantes raras que serían descartadas si se analizara cada muestra por separado.

2. Mayor precisión en muestras de baja cobertura: Si una muestra tiene solo 5 lecturas en una posición pero otras 19 muestras tienen 30 lecturas y todas muestran el mismo alelo B con alta frecuencia, la información poblacional refuerza la llamada de variante incluso en la muestra con baja cobertura.

---

# ¿Cómo implementa GATK este modelo?

**GATK (Genome Analysis Toolkit)** es un conjunto de herramientas bioinformáticas desarrollado por el Broad Institute (MIT/Harvard) para el análisis de datos de secuenciación de alto rendimiento, principalmente orientado a la detección de variantes genómicas (SNPs, INDELs, variantes estructurales). Es el estándar en genómica humana y se ha extendido ampliamente a otros organismos. Su pipeline de Best Practices es el más citado y reproducido en la literatura de genómica de poblaciones.

GATK implementa tres herramientas principalmente: 

```
HaplotypeCaller (por muestra)
    ↓
    Calcula Genotype Likelihoods para cada posición
    Genera un GVCF con bloques de referencia y sitios variantes
    
GenomicsDBImport (consolidación)
    ↓
    Combina los GVCFs de todas las muestras en una base de datos
    
GenotypeGVCFs (genotipado conjunto)
    ↓
    Estima frecuencias alélicas poblacionales
    Aplica priors de HWE
    Calcula probabilidades posteriores
    Asigna genotipos MAP a cada muestra
    Reporta QUAL, GQ (Genotype Quality) y PL (Phred-scaled likelihoods)
```
## HaplotypeCaller

HaplotypeCaller es la herramienta central del pipeline. A diferencia de los llamadores de variantes más simples que examinan posición por posición, HaplotypeCaller trabaja sobre regiones activas del genoma (ventanas del genoma donde hay evidencia de variación, es decir, donde las lecturas difieren significativamente de la referencia) y reensambla localmente los haplotipos antes de llamar variantes. Esto lo hace considerablemente más preciso en regiones complejas con múltiples variantes cercanas o INDELs.

Las regiones sin variación (no activas) son procesadas de forma simplificada y reportadas como bloques de referencia en el GVCF. En cada región activa, HaplotypeCaller construye un grafo de De Bruijn local a partir de los k-mers de las lecturas que mapean en esa región. Un grafo de De Bruijn conecta k-mers solapantes en un grafo dirigido donde cada camino posible representa un haplotipo candidato. Esto es importante porque permite a HaplotypeCaller "ver" combinaciones de variantes cercanas como haplotipos completos, en lugar de analizar cada posición de forma independiente. 

<p align="center">
 <img width="685" height="461" alt="image" src="https://github.com/user-attachments/assets/3df2e167-ce4f-40a5-b1a7-27b778b07017" />

</p>

</p>

<p align="center">
<b>Figura 1.</b> Representación esquemática de la construcción de un grafo de De Bruijn a partir de lecturas de secuenciación. Las lecturas se fragmentan en secuencias cortas de longitud k (k-mers), las cuales se conectan cuando comparten una superposición de k-1 nucleótidos. En el grafo, los nodos representan (k-1)-mers y las aristas corresponden a los k-mers. Los diferentes recorridos del grafo permiten reconstruir las secuencias presentes en la muestra, constituyendo la base de los algoritmos de ensamblaje local utilizados por herramientas como GATK HaplotypeCaller.
  
</p>

### ¿Por qué se utilizan grafos de De Bruijn?

Antes de la introducción de los grafos de De Bruijn, muchos algoritmos de ensamblaje utilizaban grafos de solapamiento (Overlap Graphs). En estos grafos, cada nodo representa una lectura completa y las aristas indican que dos lecturas se solapan. Reconstruir la secuencia original equivale a encontrar un camino o ciclo hamiltoniano, es decir, un recorrido que visite cada vértice exactamente una vez. Por lo que su resolución es muy costosa cuando existen millones de lecturas.

Los grafos de De Bruijn reformulan el problema. En lugar de representar las lecturas completas, los nodos corresponden a los (k−1)-mers y las aristas a los k-mers. La reconstrucción de la secuencia consiste entonces en encontrar un camino o ciclo euleriano, que recorre cada arista exactamente una vez. A diferencia del problema hamiltoniano, el recorrido euleriano puede resolverse mediante algoritmos eficientes en tiempo polinómico, lo que hizo posible el ensamblaje de grandes genomas y el desarrollo de herramientas modernas de análisis de secuencias.

En teoría de grafos, una arista es la conexión que une dos nodos o vértices. En un grafo de De Bruijn, los nodos representan secuencias de longitud k−1 (denominadas (k−1)-mers), mientras que las aristas representan los k-mers obtenidos de las lecturas de secuenciación. Cada arista conecta dos nodos porque el k-mer comparte un prefijo y un sufijo de longitud k−1. Por ejemplo, el k-mer ATG tiene como prefijo AT y como sufijo TG, por lo que se representa como una arista que conecta el nodo AT con el nodo TG. De esta manera, las aristas describen las posibles transiciones entre fragmentos consecutivos de la secuencia y permiten reconstruir los haplotipos o la secuencia original recorriendo el grafo.

### Algoritmo PairHMM (Pair Hidden Markov Model)

Una vez construidos los haplotipos candidatos, HaplotypeCaller realinea cada lectura de la región activa a todos los haplotipos usando el algoritmo PairHMM (Pair Hidden Markov Model). PairHMM calcula la probabilidad de que cada lectura haya sido generada por cada haplotipo candidato, considerando las probabilidades de sustitución, inserción y deleción en cada posición. Este es el paso más costoso computacionalmente.

A diferencia del alineamiento inicial contra el genoma de referencia, este paso compara las lecturas con secuencias que representan las diferentes variantes candidatas presentes en la región, permitiendo determinar cuál de los haplotipos explica mejor los datos observados.

PairHMM es un modelo probabilístico basado en Modelos Ocultos de Markov (Hidden Markov Models, HMM), el modelo evalúa la probabilidad de que una lectura haya sido generada a partir de un determinado haplotipo considerando tanto la secuencia observada como los posibles errores introducidos durante la secuenciación. Para ello, el algoritmo contempla tres tipos principales de eventos: coincidencias o sustituciones (Match/Mismatch), inserciones (Insertion) y deleciones (Deletion). Cada uno de estos eventos posee una probabilidad de transición y una probabilidad de emisión, calculadas a partir de las puntuaciones de calidad de las bases (Phred) y de modelos empíricos de error.

Durante el proceso, PairHMM recorre simultáneamente la lectura y el haplotipo mediante programación dinámica, calculando para cada posición la probabilidad acumulada de las diferentes trayectorias posibles. El resultado final es una probabilidad de alineamiento (read likelihood), que indica qué tan probable es que esa lectura provenga de cada uno de los haplotipos candidatos. Este procedimiento se repite para todas las lecturas y todos los haplotipos de la región activa, generando una matriz de probabilidades que constituye la base para el cálculo posterior de los genotipos.

Debido a que cada lectura debe compararse contra todos los haplotipos candidatos y cada comparación requiere recorrer ambas secuencias posición por posición mediante programación dinámica, PairHMM representa la etapa de mayor costo computacional dentro de HaplotypeCaller. Por esta razón, las versiones modernas de GATK incorporan optimizaciones como la vectorización mediante instrucciones SIMD (AVX) y otras mejoras de hardware, reduciendo considerablemente el tiempo de ejecución sin afectar la precisión del llamado de variantes.

A partir de los scores de PairHMM, HaplotypeCaller calcula los Genotype Likelihoods para cada posición variable dentro de la región activa. Estos likelihoods reflejan la probabilidad de los datos bajo cada genotipo posible.

Finalmente el modo -ERC GVCF, HaplotypeCaller genera un GVCF (Genomic VCF) que contiene: Sitios variantes: posiciones donde se detectó al menos un alelo alternativo, con sus Genotype Likelihoods en el campo PL y bloques de referencia: regiones consecutivas donde el individuo es homocigoto referencia con alta confianza, comprimidas en un solo registro con ALT=<NON_REF> e INFO/END indicando el fin del bloque.

---

## Generación de archivos GVCF - Genotipado conjunto 

En lugar de producir directamente un archivo VCF tradicional, HaplotypeCaller puede ejecutarse en modo **GVCF (Genomic Variant Call Format)**. El archivo GVCF almacena información tanto de posiciones variantes como no variantes, permitiendo posteriormente combinar múltiples individuos sin necesidad de repetir el llamado de variantes. Este enfoque constituye la base del genotipado conjunto utilizado por GATK. **GenomicsDBImport** es la herramienta que organiza la información de todos los GVCF en una unica base de datos optimizada para consultas rápidas y procesamiento paralelo.

GATK implementa una estrategia denominada **Joint Genotyping** o **genotipado conjunto**, mediante la cual todas las muestras son evaluadas simultáneamente utilizando un único modelo probabilístico bayesiano. En lugar de llamar variantes de forma independiente en cada individuo, cada muestra se procesa inicialmente con HaplotypeCaller en modo GVCF, generando un archivo, los archivos GVCF de todas las muestras se combinan y son analizados conjuntamente mediante la herramienta GenotypeGVCFs, la cual integra la información de todos los individuos para determinar el genotipo más probable en cada sitio.

Cada GVCF contiene, para cada posición del genoma, las probabilidades de los distintos genotipos posibles calculadas previamente por HaplotypeCaller a partir del ensamblaje local y del algoritmo PairHMM. En lugar de volver a procesar las lecturas de secuenciación, GenotypeGVCFs integra estas probabilidades entre todas las muestras y aplica un único modelo probabilístico para determinar cuáles sitios son realmente polimórficos y cuál es el genotipo más probable de cada individuo en cada posición.

Todos los individuos son evaluados exactamente en las mismas posiciones genómicas. Incluso si una muestra no presenta evidencia suficiente para llamar una variante de manera independiente, el algoritmo asigna un genotipo utilizando la información combinada de toda la cohorte. Así, una variante con baja cobertura en un individuo puede ser confirmada si también está presente en otros individuos de la población. Esto reduce la cantidad de datos faltantes (missing genotypes) y mejora la estimación de las frecuencias alélicas y genera un único archivo VCF multimuestreo consistente para todos los individuos.

Al conservar información sobre los sitios no variantes, el GVCF permite generar VCFs que incluyen tanto sitios variables como invariantes, lo cual es fundamental para calcular correctamente estadísticas de genética de poblaciones. Herramientas como **Pixy** utilizan esta información para estimar parámetros como la diversidad nucleotídica (π) y la divergencia entre poblaciones (dXY) sin introducir sesgos derivados de la ausencia de sitios invariantes.

---

# Filtrado de variantes

El llamado inicial de variantes está diseñado para maximizar la sensibilidad del análisis. En consecuencia, el conjunto de variantes obtenido suele contener tanto variantes verdaderas como falsos positivos derivados de errores de secuenciación, alineamiento o preparación de bibliotecas.

Por esta razón, el filtrado constituye una etapa indispensable del análisis bioinformático. El objetivo consiste en eliminar variantes poco confiables conservando únicamente aquellas respaldadas por suficiente evidencia experimental y estadística. Las estrategias más utilizadas son **Hard Filtering** y **Variant Quality Score Recalibration (VQSR)**.

El Hard Filtering aplica umbrales fijos sobre diferentes métricas de calidad calculadas durante el llamado de variantes. Variant Quality Score Recalibration (VQSR) por su parte es un método de filtrado basado en aprendizaje estadístico que permite distinguir variantes verdaderas de falsos positivos utilizando la información conjunta de múltiples métricas de calidad. A diferencia del Hard Filtering, que evalúa cada métrica por separado mediante umbrales fijos, VQSR considera simultáneamente variables como QD, MQ, FS, SOR, MQRankSum, ReadPosRankSum y otras anotaciones generadas durante el llamado de variantes. El objetivo es identificar el patrón de características que presentan las variantes reales y diferenciarlo del patrón asociado a errores de secuenciación, alineamiento o preparación de bibliotecas.

VQSR emplea un modelo de mezcla de distribuciones gaussianas (Gaussian Mixture Model, GMM), el cual es capaz de representar la existencia de varios grupos de variantes con características estadísticas diferentes, en lugar de asumir que todas siguen una única distribución. Para entrenar este modelo, GATK utiliza un conjunto de variantes de alta confianza (training set), como HapMap, Omni o Mills en el caso del genoma humano. A partir de estas variantes, el algoritmo aprende la distribución conjunta de las diferentes anotaciones de calidad, construyendo un espacio multidimensional en el que cada eje representa una métrica distinta. En este espacio, las variantes verdaderas tienden a agruparse formando regiones con características similares, mientras que los artefactos ocupan regiones diferentes.

En lugar de evaluar cada métrica por separado, el algoritmo estima la probabilidad de que la combinación completa de anotaciones observada para una variante sea consistente con el comportamiento de las variantes de alta confianza. A partir de estas probabilidades, calcula una puntuación denominada VQSLOD (Variant Quality Score Log-Odds), definida como el logaritmo de la razón entre la probabilidad de que la variante pertenezca al conjunto de variantes verdaderas y la probabilidad de que corresponda a un artefacto técnico. Un valor positivo de VQSLOD indica que la variante presenta un perfil de calidad similar al de las variantes utilizadas para entrenar el modelo y, por tanto, tiene una alta probabilidad de ser real. En contraste, valores negativos sugieren que la combinación de métricas observada se asemeja más a la de errores de secuenciación o alineamiento, por lo que la variante es más susceptible de ser filtrada.

Debido a que VQSR necesita un gran número de variantes y un conjunto de entrenamiento de alta calidad para construir un modelo robusto, este método se recomienda principalmente para proyectos de secuenciación de gran escala en organismos modelo, como el genoma humano. En contraste, en especies no modelo o estudios con pocas muestras, donde no existen bases de datos confiables de variantes o el número de variantes es insuficiente para entrenar el modelo, suele emplearse **Hard Filtering** como estrategia alternativa.

| Métrica | ¿Qué evalúa? | ¿Cómo se calcula? | Interpretación biológica | Valores generalmente deseables* |
|----------|--------------|-------------------|--------------------------|---------------------------------|
| **QD (Quality by Depth)** | Calidad de la variante normalizada por la cobertura. | \(QD = \frac{QUAL}{DP}\), donde **QUAL** es la calidad Phred de la variante y **DP** la profundidad de cobertura. | Permite distinguir variantes respaldadas por evidencia consistente de aquellas cuya calidad depende únicamente de una cobertura muy alta. | > 2.0 |
| **FS (Fisher Strand Bias)** | Sesgo entre las lecturas de la hebra directa y reversa. | Se construye una tabla de contingencia (Referencia/Alternativo × Forward/Reverse) y se aplica el **Test Exacto de Fisher**. El valor *p* se transforma a escala Phred: \(FS=-10\log_{10}(p)\). | Valores altos indican que el alelo alternativo aparece preferentemente en una sola hebra, lo cual suele asociarse con errores de secuenciación o alineamiento. | < 60 (SNPs)<br>< 200 (Indels) |
| **SOR (Strand Odds Ratio)** | Sesgo de hebra mediante una medida robusta basada en la razón de probabilidades. | Calcula una **Odds Ratio** entre las lecturas forward y reverse que soportan el alelo de referencia y el alternativo, aplicando posteriormente una transformación logarítmica. | Es menos sensible que FS a coberturas muy elevadas y complementa la evaluación del sesgo de hebra. | < 3.0 (SNPs)<br>< 10.0 (Indels) |
| **MQ (Mapping Quality)** | Calidad promedio del alineamiento de las lecturas que soportan la variante. | Promedio de los valores **MAPQ** asignados por el alineador (BWA-MEM, Bowtie2, Minimap2, etc.), donde \(MAPQ=-10\log_{10}(P_{alineamiento\ incorrecto})\). | Valores bajos sugieren regiones repetitivas, alineamientos ambiguos o baja confianza en la posición de las lecturas. | > 40 |
| **MQRankSum** | Diferencia en la calidad de alineamiento entre las lecturas con el alelo de referencia y las que contienen el alelo alternativo. | Aplica un **Wilcoxon Rank Sum Test (Mann–Whitney U)** sobre los valores MAPQ de ambos grupos y reporta un estadístico Z. | Valores muy negativos indican que las lecturas con el alelo alternativo presentan peor alineamiento, lo que puede reflejar errores de mapeo. | > -12.5 |
| **ReadPosRankSum** | Distribución de la posición del alelo dentro de las lecturas. | Utiliza un **Wilcoxon Rank Sum Test** para comparar la posición del alelo de referencia y del alternativo dentro de las lecturas. | Las variantes reales suelen distribuirse a lo largo de toda la lectura; los errores de secuenciación suelen concentrarse en los extremos. | > -8.0 |
| **DP (Depth)** | Profundidad de cobertura. | Conteo directo del número de lecturas que cubren la posición genómica. | Coberturas bajas disminuyen la confianza en el llamado de variantes, mientras que coberturas extremadamente altas pueden indicar regiones repetitivas, duplicados o amplificaciones locales. | Depende del experimento |
| **QUAL** | Confianza global en la existencia de la variante. | Calculada por GATK como una puntuación Phred derivada de la probabilidad de que exista una variante en esa posición. | Valores altos indican mayor evidencia estadística de que la variante es real. Sin embargo, aumenta con la cobertura, por lo que normalmente se interpreta junto con QD. | Cuanto mayor, mejor |

---

# 4.18 Formato Variant Call Format (VCF)

Una vez finalizado el llamado y filtrado de variantes, los resultados se almacenan en archivos **VCF (Variant Call Format)**, considerados el estándar internacional para representar variantes genómicas. El formato VCF fue desarrollado para facilitar el intercambio de información entre diferentes herramientas bioinformáticas y contiene tanto información de cada variante como los genotipos de todos los individuos analizados.

<p align="center">
 <img width="1504" height="619" alt="image" src="https://github.com/user-attachments/assets/97de2cec-d79a-4b14-9fe1-c58cc52e5472" />

</p>

</p>

<p align="center">
<b>Figura 2.</b> The de facto file format for storing genetic variation is the Variant Call Format (VCF) and was developed under the 1000 Genomes Project. Currently, the Large Scale Genomics work stream of the Global Alliance for Genomics & Health (GA4GH) maintain the specification of the VCF (and other high-throughput sequencing data formats). 
  
</p>

---

## 4.18.1 Estructura general

Un archivo VCF está compuesto por dos secciones principales.

### Cabecera

Incluye:

- versión del formato;
- programa utilizado;
- filtros aplicados;
- definición de todos los campos INFO y FORMAT;
- información del genoma de referencia.

---

### Registros

Cada línea representa una variante individual.

Los campos obligatorios son:

| Campo | Descripción |
|--------|-------------|
| CHROM | Cromosoma |
| POS | Posición |
| ID | Identificador |
| REF | Alelo de referencia |
| ALT | Alelo alternativo |
| QUAL | Calidad |
| FILTER | Estado del filtrado |
| INFO | Información adicional |
| FORMAT | Campos del genotipo |
| SAMPLE | Información por individuo |

---

## 4.18.2 Campo INFO

El campo INFO almacena información global de la variante.

Algunos parámetros frecuentes son:

- AC: número de alelos alternativos;
- AF: frecuencia del alelo alternativo;
- AN: número total de alelos;
- DP: profundidad total;
- MQ: calidad promedio del alineamiento;
- QD: calidad normalizada por profundidad.

---

## 4.18.3 Campo FORMAT

Describe la información reportada para cada individuo.

Entre los campos más importantes se encuentran:

### GT

Genotipo.

Ejemplos:

```
0/0
```

Homocigoto referencia.

```
0/1
```

Heterocigoto.

```
1/1
```

Homocigoto alternativo.

---

### AD

Número de lecturas que soportan cada alelo.

Ejemplo:

```
18,17
```

18 lecturas apoyan el alelo de referencia y 17 el alternativo.

---

### DP

Profundidad total de cobertura para ese individuo.

---

### GQ

Genotype Quality.

Representa la confianza estadística del genotipo asignado.

---

### PL

Likelihoods del genotipo expresadas en escala Phred.

Estos valores permiten comparar probabilísticamente los diferentes genotipos posibles.

---

# 4.19 Herramientas alternativas para el llamado de variantes

Aunque GATK constituye uno de los estándares actuales, existen otros llamadores ampliamente utilizados dependiendo del tipo de datos y los objetivos del estudio.

## bcftools

bcftools utiliza modelos probabilísticos derivados de SAMtools y destaca por:

- rapidez;
- bajo consumo de memoria;
- facilidad de uso;
- excelente integración con SAMtools.

Es ampliamente utilizado para proyectos pequeños y análisis exploratorios.

---

## FreeBayes

FreeBayes emplea un modelo basado en haplotipos que permite detectar variantes complejas y analizar organismos con diferentes niveles de ploidía.

Resulta particularmente útil en estudios de genética de poblaciones y organismos no modelo.

---

## DeepVariant

DeepVariant, desarrollado por Google, incorpora técnicas de aprendizaje profundo para el llamado de variantes.

El algoritmo transforma los alineamientos en representaciones similares a imágenes y utiliza redes neuronales convolucionales para clasificar variantes.

Diversos estudios han demostrado que DeepVariant alcanza precisiones comparables o superiores a los métodos tradicionales, especialmente en regiones genómicas complejas.

---

## Comparación general

| Herramienta | Modelo | Ventajas | Limitaciones |
|-------------|---------|----------|--------------|
| GATK | Bayesiano + PairHMM | Muy alta precisión | Mayor demanda computacional |
| bcftools | Bayesiano | Muy rápido | Menor sensibilidad en variantes complejas |
| FreeBayes | Haplotipos | Flexible para distintas ploidías | Más lento en cohortes grandes |
| DeepVariant | Aprendizaje profundo | Muy alta precisión | Requiere GPU y mayor capacidad computacional |

---


