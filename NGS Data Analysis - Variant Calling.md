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

Un archivo VCF registra la posición de cada variante con respecto a un genoma de referencia e incluye información detallada sobre los alelos observados, la calidad del llamado, las métricas de filtrado, diversas anotaciones y los genotipos de una o varias muestras. A diferencia de un archivo FASTA o BAM, que almacenan secuencias o alineamientos, un VCF resume únicamente las posiciones donde existe información relevante para el análisis de variantes.

La estructura del archivo está organizada en tres secciones principales:

Metainformación: corresponde a las primeras líneas del archivo, precedidas por el símbolo ##. Estas líneas describen la versión del formato VCF, el genoma de referencia utilizado, los programas empleados durante el llamado de variantes y la definición de las anotaciones presentes en el archivo, como los campos INFO, FORMAT y FILTER.

Cabecera: es la última línea que comienza con el carácter #. Define el nombre de las columnas del archivo y, cuando se trata de un VCF multimuestreo, incluye el identificador de cada muestra analizada.

Registros de variantes: cada línea posterior representa una variante detectada en una posición específica del genoma. Además de indicar el cromosoma, la posición y los alelos de referencia y alternativos, cada registro almacena información sobre la calidad del llamado, los filtros aplicados, diversas anotaciones calculadas durante el análisis y los genotipos de todas las muestras.

---


# 4.19 Herramientas alternativas para el llamado de variantes

Aunque GATK HaplotypeCaller constituye uno de los estándares actuales para el llamado de variantes en datos de secuenciación masiva, existen otras herramientas ampliamente utilizadas. Cada una implementa modelos estadísticos y estrategias de análisis diferentes, por lo que su rendimiento puede variar dependiendo del organismo, la profundidad de secuenciación, la ploidía y los objetivos del estudio.

## bcftools

bcftools es la evolución del antiguo llamador de variantes de SAMtools y se caracteriza por implementar un modelo probabilístico relativamente simple basado en las probabilidades de genotipo (genotype likelihoods). A partir del archivo BAM, evalúa la evidencia aportada por las lecturas en cada posición del genoma y estima el genotipo más probable utilizando un modelo bayesiano.

A diferencia de GATK HaplotypeCaller, bcftools no realiza un ensamblaje local de haplotipos. El algoritmo analiza cada posición del genoma prácticamente de manera independiente, utilizando la información proveniente del alineamiento existente. Esto reduce considerablemente el tiempo de ejecución y el consumo de memoria, aunque puede disminuir la precisión en regiones complejas o en la detección de inserciones y deleciones (indels).

Debido a su rapidez y facilidad de uso, bcftools es ampliamente empleado en proyectos pequeños, análisis exploratorios, organismos no modelo y como herramienta de control de calidad o validación de variantes.

---

## FreeBayes

FreeBayes utiliza un enfoque basado en haplotipos en lugar de evaluar cada variante de forma completamente independiente. El algoritmo identifica conjuntos de variantes cercanas presentes en las mismas lecturas y estima la probabilidad de los diferentes haplotipos utilizando un modelo bayesiano.

Aunque comparte con HaplotypeCaller la idea de considerar haplotipos completos, FreeBayes no realiza un ensamblaje local mediante grafos de De Bruijn. En su lugar, construye los haplotipos directamente a partir de las combinaciones de variantes observadas en las lecturas alineadas. Esto simplifica el proceso y reduce el costo computacional, aunque puede ser menos robusto en regiones altamente repetitivas o con variantes estructurales complejas.

Una de sus principales ventajas es que permite trabajar de manera natural con organismos de ploidía variable (diploides, poliploides o mezclas de individuos), así como con poblaciones microbianas o muestras con múltiples haplotipos. Por esta razón, FreeBayes es ampliamente utilizado en estudios de genética de poblaciones, evolución y organismos no modelo.

---

## DeepVariant

DeepVariant, desarrollado por Google, representa un cambio importante en el enfoque tradicional del llamado de variantes. En lugar de construir un modelo estadístico basado en supuestos explícitos sobre los errores de secuenciación, utiliza aprendizaje profundo (deep learning) para aprender directamente los patrones presentes en los datos.

El algoritmo transforma el alineamiento de las lecturas alrededor de cada posible variante en una representación similar a una imagen (pileup image), donde cada fila corresponde a una lectura y diferentes canales codifican información como las bases, las calidades Phred, la calidad del alineamiento, la orientación de la hebra y otras características relevantes. Estas imágenes son analizadas por una red neuronal convolucional (CNN) entrenada con millones de variantes previamente validadas.

En lugar de calcular explícitamente probabilidades mediante modelos bayesianos o algoritmos como PairHMM, la red neuronal aprende automáticamente las características que distinguen una variante verdadera de un error de secuenciación. Como resultado, DeepVariant suele alcanzar una precisión muy elevada, especialmente en regiones difíciles del genoma y en la detección de SNPs e indels. Sin embargo, requiere modelos previamente entrenados para cada tecnología de secuenciación (Google diseño modelos para Illumina, PacBio HiFi, Oxford Nanopore) y demanda mayores recursos computacionales, particularmente el uso de GPU para obtener un rendimiento óptimo.

Aunque diversos estudios han demostrado que DeepVariant puede alcanzar una precisión comparable o incluso superior a la de GATK HaplotypeCaller en la detección de SNPs e indels, GATK continúa siendo el estándar de referencia en numerosos proyectos de investigación debido a que GATK no es únicamente un llamador de variantes, sino un ecosistema completo de herramientas para el preprocesamiento de datos, recalibración de la calidad de bases (BQSR), llamado de variantes, genotipado conjunto, filtrado (Hard Filtering y VQSR) y análisis de cohortes.

Por eso, actualmente muchos proyectos aprovechan lo mejor de ambos mundos: utilizan DeepVariant para generar los llamados iniciales por muestra y posteriormente realizan el genotipado conjunto y otros análisis poblacionales con herramientas del ecosistema GATK (o con GLnexus, que está diseñado para combinar llamadas de DeepVariant). De esta forma se obtiene una alta precisión sin perder la capacidad de analizar grandes cohortes de individuos.

DeepVariant es completamente gratuito y de código abierto. El código fuente y las versiones oficiales se encuentran en GitHub (https://github.com/google/deepvariant?utm_source) 

---

# Un flujo de trabajo completo para el llamado de variantes con GATK

En este módulo se procesaron tres muestras de *Solanum sección lycopersicum* secuenciadas por WGS con Illumina: una accesión de *S. lycopersicum var. cerasiforme* (SLC; SRR31477438), una accesión de *S. lycopersicum var. lycopersicum* (LA1924; SRR38359005) y una accesión de *S. pimpinellifolium* (SRR37254991), mapeadas contra el genoma de referencia del tomate cultivado (*S. lycopersicum var. lycopersicum* Micro-Tom, ensamblaje SLM_r2.1 (GCF_036512215.1; 832.8 Mb). Las lecturas WGS de las muestras fueron descargadas desde la base de datos SRA del NCBI usando fasterq-dump, en formato FASTA.

Todos los análisis se ejecutaron en el clúster de cómputo HPC perteneciente a la Facultad de Ciencias de la Universidad Nacional de Colombia mediante scripts SLURM, utilizando el gestor de paquetes Conda y entornos virtuales asociados. La instalación y resolución de dependencias se ejecuto a través de Anaconda y Miniconda.

---

## Generación de archivos VCF de AllSites mediante GATK

Se utilizo la guía para generar archivos VCF de AllSites facilitada por Pixy utilizando GATK. Pixy es una herramienta de línea de comandos que calcula π, d xy , F ST , θ de Watterson y D de Tajima a partir de un archivo VCF. A diferencia de la mayoría de las herramientas que calculan estas estadísticas, pixy produce estimaciones insesgadas en presencia de datos faltantes (https://pixy.readthedocs.io/en/latest/about.html).

GATK recomienda identificar primero las variantes por muestra utilizando HaplotypeCaller en modo GVCF (Paso 1 a continuación). A continuación, GenomicsDBImport consolida la información de los archivos GVCF de todas las muestras para mejorar la eficiencia del genotipado conjunto (Paso 2 a continuación). En el tercer paso, GenotypeGVCFs genera un conjunto de SNP e INDEL identificados conjuntamente, listos para su filtrado y análisis. 

**NOTA:** ``--all-sites`` is not compatible with VQSR. The 0/0 (invariant) records emitted by GenotypeGVCFs --all-sites typically carry only DP in the INFO field and lack the annotations that VariantRecalibrator relies on (QD, FS, SOR, MQ, MQRankSum, ReadPosRankSum, InbreedingCoeff). For pixy input you should hard-filter instead, and — per the warning at the top of this page — filter variant and invariant sites separately before concatenating them back together.

---
Una vez completado el preprocesamiento de los archivos BAM. Se procede al llamado de variantes para cada muestra utilizando HaplotypeCaller. En esta etapa, cada individuo se analiza de forma independiente para identificar SNPs e indels mediante el ensamblaje local de haplotipos y el algoritmo PairHMM. En lugar de generar directamente un archivo VCF convencional, HaplotypeCaller se ejecuta en modo GVCF mediante la opción -ERC GVCF. Este formato conserva información tanto de los sitios variantes como de los sitios no variantes (bloques de referencia), permitiendo posteriormente realizar el genotipado conjunto (Joint Genotyping) de múltiples muestras sin necesidad de repetir el llamado de variantes.

```
# Cargar del módulo de conda
module load envs/anaconda3
source /scratchsan1/anaconda3/etc/profile.d/conda.sh
conda activate gatk-4.6.2

export REF="/scratchsan/amateusr/ref_genome_tomate/GCF_036512215.1_SLM_r2.1_genomic.fna"
export OUT="/scratchsan/amateusr/outs"

# ── Paso 1: HaplotypeCaller por cada muestra

gatk --java-options "-Xmx4G" HaplotypeCaller \
    -R $REF \
    -I $OUT/SRR31477438.markdup.bam \
    -O $OUT/SRR31477438.g.vcf.gz \
    -ERC GVCF \
    --native-pair-hmm-threads 8

# Se mantuvieron los mismos parametros para cada una de las tres muestras

```
Para cada muestra se obtiene un archivo:

SRR31477438.g.vcf.gz
SRR38359005.g.vcf.gz
SRR37254991.g.vcf.gz

Estos archivos contienen las probabilidades de genotipo calculadas para todas las posiciones del genoma y constituyen la entrada para la etapa de Joint Genotyping, en la cual se combinará la información de todas las muestras para generar un único archivo VCF multimuestreo con los genotipos finales.

---

Una vez generado el archivo GVCF para cada muestra, el siguiente paso consiste en consolidarlos mediante la herramienta GenomicsDBImport. Esta herramienta no realiza el genotipado conjunto ni modifica los genotipos previamente calculados. Su función es importar la información contenida en múltiples archivos GVCF y almacenarla en una base de datos GenomicsDB, una estructura optimizada para acceder de manera eficiente a la información genómica durante el proceso de Joint Genotyping.

Para realizar la importación, GenomicsDBImport requiere especificar los intervalos genómicos que serán procesados mediante la opción -L. Estos intervalos pueden corresponder a cromosomas completos, regiones específicas o, como en este ejemplo, a todos los contigs presentes en el genoma de referencia. El archivo de intervalos se genera automáticamente a partir del índice FASTA (.fai) utilizando el siguiente comando:

```

awk '{print $1":1-"$2}' $REF.fai > $OUT/intervals.list

```

Este comando extrae el nombre y la longitud de cada contig del archivo índice y construye un intervalo que abarca completamente cada secuencia del genoma de referencia. Posteriormente, todos los archivos GVCF son importados a la base de datos GenomicsDB:

```

gatk --java-options "-Xmx4G" GenomicsDBImport \
    -V $OUT/SRR31477438.g.vcf.gz \
    -V $OUT/SRR38359005.g.vcf.gz \
    -V $OUT/SRR37254991.g.vcf.gz \
    --genomicsdb-workspace-path $OUT/allsamples_genomicsdb \
    -L $OUT/intervals.list

```

La ejecución no produce un archivo VCF, sino un directorio denominado:

allsamples_genomicsdb/

Este directorio contiene la base de datos GenomicsDB, en la cual la información de todas las muestras queda organizada por posiciones genómicas. Esta estructura permite que la siguiente herramienta, GenotypeGVCFs, consulte rápidamente la información de cada región del genoma sin tener que abrir y recorrer todos los archivos GVCF de forma independiente.

---

Una vez consolidados los archivos GVCF en la base de datos GenomicsDB, se realiza el genotipado conjunto (Joint Genotyping) utilizando la herramienta GenotypeGVCFs. GenotypeGVCFs consulta la información contenida en GenomicsDB, integra simultáneamente la evidencia de todas las muestras y aplica un modelo bayesiano para identificar los alelos presentes en cada posición del genoma y asignar el genotipo más probable a cada individuo.

El comando utilizado fue:

```
gatk --java-options "-Xmx4G" GenotypeGVCFs \
    -R $REF \
    -V gendb://$OUT/allsamples_genomicsdb \
    --all-sites \
    -L $OUT/intervals.list \
    -O $OUT/allsamples_allsites.vcf.gz

```

La opción --all-sites fue utilizada para generar un VCF que conserva tanto los sitios variantes como los invariantes, requisito indispensable para el cálculo correcto de estadísticas como la diversidad nucleotídica (π) y la divergencia entre poblaciones (d<sub>XY</sub>) mediante la herramienta Pixy. A diferencia de otros programas, Pixy utiliza explícitamente la información de los sitios invariantes para estimar el número total de posiciones comparables entre individuos, evitando el sesgo que se produce cuando únicamente se analizan posiciones polimórficas.

Archivo generado

Como resultado se obtiene un archivo:

allsamples_allsites.vcf.gz

Se identificaron 8,729,943 SNPs en el VCF conjunto de las muestras sobre un genoma de aproximadamente 833 Mb, lo que corresponde a una densidad de 10.5 SNPs por kb. Este valor es consistente con la divergencia genética esperada entre una accesiones silvestres de Solanum sección lycopersicum y una variedad cultivada de S. lycopersicum var lycopersicum. La elevada cantidad de polimorfismos refleja la acumulación de diferencias genéticas entre los linajes a lo largo de su historia evolutiva. Aunque la domesticación del tomate produjo una reducción de la diversidad genética mediante cuellos de botella poblacionales y selección artificial, esta constituye solo uno de los procesos responsables de la divergencia observada. La variación detectada también es consecuencia de la historia evolutiva previa de las poblaciones silvestres, la acumulación de mutaciones, la deriva genética y los procesos de selección natural y artificial que han actuado sobre ambos linajes.

---

El conjunto inicial de variantes aún puede incluir falsos positivos, por esta razón, en el siguiente módulo se abordará el filtrado del archivo VCF, empleando criterios de calidad para conservar únicamente variantes de alta confianza y se describirá el cálculo de estadísticas de genética de poblaciones utilizando Pixy.

---

# Referencias

- DePristo, M. A., et al. (2011). A framework for variation discovery and genotyping using next-generation DNA sequencing data. Nature Genetics, 43(5), 491–498.
- Li, H., & Durbin, R. (2009). Fast and accurate short read alignment with Burrows-Wheeler Aligner. Bioinformatics, 25(14), 1754–1760.
- McKenna, A., et al. (2010). The Genome Analysis Toolkit: A MapReduce framework for analyzing next-generation DNA sequencing data. Genome Research, 20(9), 1297–1303.
- Picard toolkit. (2019). Broad Institute. https://broadinstitute.github.io/picard/
- Poplin, R., et al. (2018). Scaling accurate genetic variant discovery to tens of thousands of samples. bioRxiv. https://doi.org/10.1101/201178
- Takei, H., et al. (2021). De novo genome assembly of two tomato ancestors, Solanum pimpinellifolium and Solanum lycopersicum var. cerasiforme, by long-read sequencing. DNA Research, 28(1), dsaa029.
- Teterina, A. A., et al. (2024). pixy: Unbiased estimation of nucleotide diversity and divergence in the presence of missing data. Molecular Ecology Resources, 21(8), 2759–2764.



