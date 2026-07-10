# NGS Data Analysis - Mapping

> Una guía sobre los fundamentos del mapeo de lecturas y un flujo de trabajo completo con datos de secuenciación de nueva generación (Next Generation Sequencing, NGS).

---

# Introducción

Las tecnologías de secuenciación de nueva generación (NGS) revolucionaron la biología molecular al permitir la obtención simultánea de millones de fragmentos de ADN en una única corrida de secuenciación. A diferencia del método clásico de Sanger, que produce una única secuencia por reacción, las plataformas NGS generan millones o incluso miles de millones de lecturas (reads), reduciendo considerablemente el costo por base secuenciada y aumentando la velocidad de adquisición de datos. El enorme volumen de información generado por estas tecnologías requiere algoritmos computacionales especializados capaces de transformar millones de lecturas cortas en información biológicamente interpretable. 

Actualmente, plataformas de secuenciación como Illumina, Oxford Nanopore Technologies y Pacific Biosciences permiten la optención de millones de pequeñas secuencias independientes denominadas **lecturas** (*reads*), cuya longitud puede variar desde aproximadamente 50 hasta varios miles de nucleótidos dependiendo de la tecnología utilizada. Estas lecturas representan únicamente fragmentos del ADN original, por lo que deben ser procesadas computacionalmente para reconstruir la información biológica del genoma. El análisis bioinformático transforma estos datos crudos en información útil mediante una serie de etapas secuenciales que incluyen:

1. Control de calidad de las lecturas.
2. Eliminación de adaptadores y secuencias de baja calidad.
3. Alineamiento contra un genoma de referencia.
4. Procesamiento de los alineamientos.
5. Llamado de variantes.
6. Filtrado de variantes.
7. Anotación funcional.

Cada una de estas etapas reduce progresivamente la incertidumbre asociada a los datos experimentales, permitiendo identificar con mayor precisión las diferencias genéticas reales entre individuos.

En este módulo se describen los fundamentos teóricos del alineamiento de lecturas contra un genoma de referencia y del llamado de variantes en datos de secuenciación de nueva generación (NGS). Se abordan los principios biológicos, computacionales y estadísticos que sustentan estas etapas y se presenta un flujo de trabajo completo para el análisis de datos NGS a partir de datos de *Solanum pimpinellifolium, Solanum lycopersicum var. cerasiforme y Solanum lycopersicum var. lycopersicum.*

---

# Alineamiento de lecturas contra un genoma de referencia (Reference-based alignment o Read mapping).

El alineamiento de secuencias posee aplicaciones en prácticamente todas las áreas de la biología molecular y constituye uno de los problemas fundamentales de la bioinformática. Consiste en establecer la correspondencia óptima entre dos o más secuencias de ADN, ARN o proteínas con el propósito de identificar regiones conservadas y diferencias evolutivas. Desde el punto de vista computacional, el alineamiento busca maximizar la similitud entre secuencias permitiendo la aparición de sustituciones, inserciones y deleciones cuando estas representan mejor la historia evolutiva de las moléculas comparadas. 

El alineamiento de millones de lecturas representa por tanto un problema computacional extremadamente complejo. Una secuenciación puede generar más de 500 millones de lecturas. Comparar cada una de ellas contra todas las posiciones posibles de un genoma de aproximadamente 900 Mb, como el de *Solanum lycopersicum*, requeriría un tiempo computacional prohibitivo si se utilizaran algoritmos clásicos de comparación exhaustiva. Por ello, los alineadores modernos emplean estructuras de datos comprimidas y algoritmos heurísticos que permiten acelerar considerablemente el proceso sin comprometer significativamente la precisión.

El objetivo del alineamiento **read mapping**, consiste en localizar la posición más probable del genoma donde se originó cada lectura generada por el secuenciador. La correcta ubicación de las lecturas resulta indispensable para todas las etapas posteriores del análisis. Si una lectura es alineada en una posición incorrecta, cualquier diferencia observada respecto al genoma de referencia podría interpretarse erróneamente como una variante genética cuando en realidad corresponde a un error de alineamiento. Por esta razón, el alineamiento constituye el paso crítico sobre cualquier estudio.

<p align="center">
 <img width="782" height="455" alt="image" src="https://github.com/user-attachments/assets/8cd124c2-0071-48d7-b2a6-349354d9e67a" />

</p>

</p>

<p align="center">
<b>Figura 1.</b> Alineamiento de lecturas (reads) de secuenciación frente a una secuencia de referencia. La superposición de múltiples lecturas genera una cobertura que permite identificar variantes mediante la comparación de las bases presentes en cada posición del genoma. Este procedimiento constituye la base del variant calling en análisis de secuenciación de nueva generación (NGS).
  
</p>

---
# Algoritmos modernos de alineamiento

El alineamiento contra referencia consiste en asignar cada lectura de ADN a la región del genoma donde probablemente fue originada. Durante este proceso, el alineador compara la secuencia de cada lectura con millones de posiciones posibles del genoma de referencia y calcula cuál de ellas representa el alineamiento más probable.

## Alineamiento de lecturas cortas

Los alineadores actuales emplean una estrategia denominada **Seed-and-Extend**, diseñada para acelerar el proceso de búsqueda. En lugar de comparar la lectura completa contra todas las posiciones del genoma, el algoritmo divide inicialmente cada lectura en pequeños fragmentos llamados **semillas** (*seeds - k-mers*). Cada una de estas semillas se busca rápidamente utilizando el FM-index. Una vez encontrada una coincidencia potencial, el algoritmo extiende el alineamiento hacia ambos extremos utilizando técnicas de programación dinámica. Para esta fase de extensión se utilizan variantes optimizadas del algoritmo de Smith-Waterman, el cual permite identificar el alineamiento local de mayor puntuación considerando coincidencias, sustituciones, inserciones y deleciones.

La estrategia Seed-and-Extend representa actualmente la base computacional de la mayoría de los alineadores modernos utilizados en genómica, incluyendo **Bowtie2, BWA-MEM** y otros algoritmos ampliamente empleados en proyectos de resecuenciación. Este enfoque presenta múltiples ventajas: disminuye considerablemente el tiempo de ejecución; reduce el consumo de memoria; permite detectar pequeñas inserciones y deleciones; mejora la precisión del alineamiento en regiones variables y facilita el procesamiento de millones de lecturas en tiempos razonables.

El funcionamiento interno de los principales alineadores de lecturas cortas incluye:

### **1. Construcción del índice:** 

Antes de iniciar el alineamiento, el genoma de referencia es transformado mediante la Burrows-Wheeler Transform y posteriormente indexado utilizando el FM-index. **La Transformada de Burrows-Wheeler (Burrows-Wheeler Transform, BWT)** constituye una transformación reversible que reorganiza los caracteres del genoma para agrupar regiones similares sin modificar la información original. Inicialmente fue desarrollada para algoritmos de compresión de datos, pero posteriormente se demostró que esta transformación permite realizar búsquedas extremadamente eficientes sobre secuencias biológicas. Sobre la BWT se construye el denominado **FM-index**, una estructura de datos que permite localizar rápidamente cualquier subsecuencia dentro del genoma sin necesidad de recorrerlo completamente. Gracias al FM-index, herramientas como Bowtie2 y BWA pueden localizar millones de lecturas utilizando únicamente unos pocos gigabytes de memoria RAM, incluso para genomas de gran tamaño. La combinación BWT–FM-index constituye actualmente el estándar para el alineamiento de lecturas cortas. Esta etapa solamente debe ejecutarse una vez para cada genoma.

### **2. Selección de semillas:** 

Cada lectura es dividida en pequeños fragmentos (seeds). Las semillas corresponden a pequeños kmers cuya longitud varia. Estas semillas son buscadas dentro del índice del genoma. El valor k, que corresponde a la longitud, expresada en número de nucleótidos, de las subsecuencias o k-mers utilizadas como semillas (seeds) durante el proceso de búsqueda. Es uno de los parámetros fundamentales en los algoritmos de alineamiento. En lugar de comparar una lectura completa contra todas las posiciones del genoma de referencia, el alineador la divide en pequeños fragmentos de longitud k. Cada uno de estos fragmentos se busca rápidamente en el índice del genoma mediante estructuras de datos como el FM-index, permitiendo identificar regiones candidatas donde posteriormente se realiza el alineamiento completo mediante la estrategia seed-and-extend.

La elección del tamaño del k-mer representa un compromiso entre sensibilidad y velocidad del alineamiento: Valores pequeños de k generan un mayor número de coincidencias potenciales, incrementando la sensibilidad del algoritmo y permitiendo detectar variantes o errores de secuenciación. Sin embargo, también aumentan el tiempo de ejecución y la probabilidad de obtener alineamientos ambiguos, especialmente en regiones repetitivas. Valores grandes de k producen menos coincidencias, acelerando el proceso de alineamiento y reduciendo el número de falsos emparejamientos. No obstante, disminuyen la sensibilidad cuando las lecturas contienen errores o múltiples variantes respecto al genoma de referencia. Por esta razón, los alineadores no utilizan un único valor fijo de k, sino estrategias adaptativas que optimizan automáticamente la longitud y el número de semillas según las características de las lecturas y del genoma analizado.

La principal diferencia entre ambos alineadores **(Bowtie2 y BWA-MEM)** radica en la selección de las semillas: Bowtie2 utiliza semillas de longitud fija, mientras que BWA-MEM emplea Maximal Exact Matches (MEMs), lo que generalmente mejora el rendimiento en lecturas más largas y regiones genómicas complejas. 

A diferencia de Bowtie2, que divide las lecturas en semillas (*seeds*) de longitud fija, **BWA-MEM** a diferencia de Bowtie2, inicia el proceso de alineamiento identificando las **Maximal Exact Matches (MEM)**, las coincidencias exactas de mayor longitud compartidas entre una lectura y el genoma de referencia. Una MEM corresponde al segmento continuo más largo que coincide exactamente entre ambas secuencias y que no puede extenderse ni hacia la izquierda ni hacia la derecha sin introducir una discrepancia.

El algoritmo explora el índice del genoma, construido mediante la Transformada de Burrows-Wheeler (BWT) y el FM-index, para localizar estas coincidencias máximas. Una vez identificadas, las MEM se utilizan como puntos de anclaje (*anchors*) para iniciar el proceso de extensión del alineamiento. El uso de MEM presenta varias ventajas respecto al empleo de semillas de longitud fija. Al utilizar las coincidencias exactas más largas disponibles, el algoritmo reduce el número de alineamientos candidatos que deben evaluarse, disminuye la probabilidad de obtener alineamientos ambiguos en regiones repetitivas y mejora la precisión del mapeo, especialmente en lecturas de mayor longitud. Además, esta estrategia incrementa la eficiencia computacional al concentrar el esfuerzo de alineamiento únicamente en las regiones del genoma con mayor probabilidad de corresponder al origen de la lectura.

Por estas razones, BWA-MEM se ha convertido en uno de los alineadores más utilizados para datos de secuenciación del genoma completo (Whole Genome Sequencing, WGS) y constituye el alineador recomendado por las **GATK Best Practices** para el análisis de variantes en organismos con un genoma de referencia de alta calidad.

### 3. Búsqueda rápida:

Cada seed es localizada mediante el FM-index. En esta etapa el algoritmo identifica únicamente regiones candidatas donde podría encontrarse la lectura completa. Esto reduce drásticamente el número de comparaciones necesarias. 

### 4. Extensión del alineamiento:

Una vez identificadas una o varias regiones candidatas mediante la búsqueda de semillas (*seeds*) en el índice del genoma, el alineador procede a la etapa de **extensión del alineamiento (*seed-and-extend*)**. Para ello, ambos alineadores emplean un algoritmo de programación dinámica inspirado en **Smith-Waterman**, diseñado para realizar alineamientos locales. A diferencia de una búsqueda basada únicamente en coincidencias exactas, este algoritmo calcula el alineamiento de mayor puntuación considerando diferentes eventos biológicos y tecnicos que pueden ocurrir durante la secuenciación o como consecuencia de la variación genética entre el individuo analizado y el genoma de referencia.

El algoritmo asigna una puntuación a cada posible alineamiento mediante un sistema de recompensas y penalizaciones. Las coincidencias exactas (*matches*) incrementan la puntuación total, mientras que las sustituciones (*mismatches*), inserciones (*insertions*) y deleciones (*deletions*), conocidas conjuntamente como **indels**, reciben penalizaciones proporcionales a su longitud y frecuencia esperada. De esta manera, el algoritmo identifica el alineamiento óptimo que maximiza la similitud entre la lectura y la referencia.

Otra característica importante de esta etapa es la posibilidad de realizar **alineamiento local**, permitiendo el recorte automatico (*soft clipping*) de nucleótidos localizados en los extremos de la lectura cuando presentan baja calidad o corresponden a adaptadores remanentes del proceso de secuenciación. Esto mejora la precisión del alineamiento y aumenta la probabilidad de mapear correctamente lecturas que, de otro modo, serían descartadas.

<p align="center">
  <img width="850" height="550" alt="image" src="https://github.com/user-attachments/assets/fd49b62e-f5ff-420f-ab00-050da39c6de3" />

</p>

<p align="center">
<b>Figura 2.</b> Esquema general de la estrategia seed-and-extend utilizada por alineadores modernos como Bowtie2 y BWA-MEN. La estructura de datos corresponde al FM-index, construido a partir de la Transformada de Burrows-Wheeler (BWT), el cual permite localizar rápidamente las posiciones de cada semilla dentro del genoma de referencia. 
  
</p>

---

## 5. Evaluación del alineamiento y Formatos SAM/BAM:

Una vez finalizado el proceso de alineamiento, herramientas como Bowtie2 o BWA-MEM generan como salida un archivo en formato SAM (Sequence Alignment/Map). Cada registro del archivo corresponde a una lectura alineada e incluye información calculada directamente por el alineador, como la posición de mapeo, la calidad del alineamiento (MAPQ), la cadena CIGAR, que describe cómo se alineó la lectura respecto al genoma de referencia, y el campo FLAG, que codifica mediante representación binaria el estado y las características del alineamiento. 

Posteriormente, el archivo SAM suele convertirse al formato binario BAM mediante herramientas como SAMtools, lo que permite reducir el espacio de almacenamiento y optimizar las operaciones de ordenamiento, indexación y análisis posteriores.

En la siguiente sección se analizan en detalle las principales métricas de evaluación del alineamiento (MAPQ, CIGAR y FLAG), así como la estructura y utilidad de los formatos SAM y BAM.

### **Mapping Quality Score (MAPQ)** 

El alineamiento obtenido es evaluado mediante un sistema de puntuación que considera la longitud del alineamiento, el número de coincidencias, las penalizaciones por indels y mismatches, así como la existencia de posibles alineamientos alternativos en otras regiones del genoma. Con esta información, el programa selecciona el mejor alineamiento para cada lectura y calcula posteriormente el **Mapping Quality Score (MAPQ)**, que representa la confianza estadística de que la lectura ha sido ubicada en la posición correcta del genoma de referencia.

El valor de MAPQ se almacena en el archivo **SAM (Sequence Alignment/Map)**, específicamente en la quinta columna de cada registro de alineamiento, y se conserva posteriormente en su versión binaria BAM. Cada línea del archivo SAM representa una lectura individual e incluye información como el nombre de la lectura, el cromosoma de alineamiento, la posición genómica, el valor de MAPQ, la cadena CIGAR y otros campos que describen las características del alineamiento. De esta manera, el archivo SAM/BAM contiene toda la información necesaria para evaluar la calidad del mapeo y constituye el principal insumo para las etapas posteriores del análisis bioinformático.

El MAPQ se expresa mediante una escala Phred, en la que valores más altos indican una mayor probabilidad de que la lectura haya sido alineada correctamente. En términos generales, un valor de MAPQ = 0 indica que la lectura puede alinearse con una calidad similar en múltiples posiciones del genoma, mientras que valores superiores a 30 representan alineamientos de alta confianza y un valor de 60 suele corresponder a un alineamiento prácticamente único. Aunque la interpretación de esta escala es común entre los distintos alineadores, el algoritmo específico utilizado para estimar el MAPQ varía entre herramientas como Bowtie2 y BWA-MEM.

Generalmente:

| MAPQ | Interpretación |
|------|----------------|
| 0 | Múltiples posiciones posibles |
| 10-20 | Baja confianza |
| 20-30 | Confianza moderada |
| >30 | Alta confianza |
| 60 | Alineamiento prácticamente único |

Durante el llamado de variantes, el valor de MAPQ constituye uno de los principales criterios para evaluar la confiabilidad de las lecturas. Generalmente, las lecturas con valores bajos de MAPQ son descartadas o reciben un menor peso en el análisis, ya que presentan una mayor probabilidad de estar alineadas incorrectamente y, por lo tanto, de generar falsos positivos en la identificación de SNPs e inserciones o deleciones (indels).

### **La cadena CIGAR (Compact Idiosyncratic Gapped Alignment Report)**

**La cadena CIGAR (Compact Idiosyncratic Gapped Alignment Report)** es otro de los campos importantes del formato SAM/BAM, ya que describe de manera compacta cómo una lectura se alinea con respecto al genoma de referencia. Esta cadena resume todas las operaciones necesarias para representar el alineamiento, indicando las coincidencias, discrepancias, inserciones, deleciones y otras modificaciones presentes entre la lectura y la secuencia de referencia. Gracias a esta información, es posible reconstruir el alineamiento sin necesidad de comparar nuevamente la lectura con el genoma.

Cada operación de la cadena CIGAR está representada por un número seguido de una letra. El número indica la cantidad de nucleótidos involucrados y la letra especifica el tipo de operación realizada. Entre las operaciones más utilizadas se encuentran M, que representa posiciones alineadas y puede incluir tanto coincidencias (matches) como sustituciones (mismatches); I, que indica una inserción de nucleótidos en la lectura con respecto al genoma de referencia; y D, que representa una deleción en la lectura respecto a la referencia.

Asimismo, la operación N indica una región omitida del genoma, utilizada principalmente en alineamientos de RNA-Seq para representar intrones. La letra S (soft clipping) señala que determinados nucleótidos de los extremos de la lectura fueron excluidos del alineamiento, aunque permanecen almacenados en el archivo SAM. En cambio, H (hard clipping) elimina completamente esos nucleótidos del registro del alineamiento. Finalmente, los símbolos = y X permiten distinguir explícitamente entre coincidencias exactas y sustituciones, respectivamente, aunque muchos alineadores continúan utilizando la operación M para representar ambas situaciones.

Por ejemplo, una cadena 76M indica que los 76 nucleótidos de la lectura fueron alineados de forma continua respecto al genoma de referencia. Una cadena 35M2I39M significa que después de los primeros 35 nucleótidos alineados existe una inserción de dos nucleótidos en la lectura, seguida por otros 39 nucleótidos alineados. Por su parte, una cadena 20S56M indica que los primeros 20 nucleótidos fueron recortados mediante soft clipping, mientras que los 56 nucleótidos restantes participaron en el alineamiento. Estas operaciones permiten representar de forma precisa eventos biológicos como inserciones y deleciones, además de corregir problemas derivados de regiones de baja calidad o adaptadores residuales presentes en las lecturas.

### **El campo FLAG**

**El campo FLAG** es un valor numérico presente en cada registro del archivo SAM/BAM que almacena información sobre el estado y las características del alineamiento de una lectura. A diferencia de otros campos, el FLAG no representa un único atributo, sino que utiliza una codificación binaria, en la cual cada bit indica una propiedad específica de la lectura. De esta manera, un único número puede contener simultáneamente múltiples características del alineamiento.

Entre las propiedades más importantes codificadas por el campo FLAG se encuentran si la lectura pertenece a un experimento paired-end, si ambas lecturas de un par fueron alineadas correctamente, si la lectura o su pareja no lograron alinearse al genoma de referencia, si la lectura se encuentra alineada sobre la hebra directa o reversa, si corresponde a un duplicado generado durante la amplificación por PCR y si el alineamiento es secundario o suplementario. Esta información permite describir completamente el estado de cada lectura dentro del análisis bioinformático.

Durante las etapas posteriores del procesamiento de datos, el campo FLAG constituye un criterio fundamental para el filtrado de lecturas. Herramientas como SAMtools, Picard y GATK utilizan estos indicadores para excluir lecturas duplicadas, alineamientos secundarios o lecturas que no cumplen los criterios mínimos de calidad, reduciendo así la probabilidad de introducir errores durante el llamado de variantes. En consecuencia, la correcta interpretación del campo FLAG contribuye a mejorar la confiabilidad del análisis y a obtener un conjunto de variantes genéticas de mayor calidad.

Un alineamiento de alta calidad constituye por tanto el requisito indispensable para el descubrimiento confiable de variantes.

---
## Formatos SAM/BAM:

El archivo **SAM** es un formato de texto plano diseñado para almacenar los resultados del alineamiento de lecturas contra un genoma de referencia. Debido a que los experimentos de secuenciación de nueva generación generan millones de lecturas, los archivos SAM pueden alcanzar tamaños de varios gigabytes, dificultando su almacenamiento y procesamiento. Por esta razón, el formato SAM suele convertirse a **BAM (Binary Alignment/Map)**, que corresponde a su representación binaria comprimida. Ambos formatos contienen exactamente la misma información; la diferencia radica en que el archivo BAM ocupa menos espacio en disco y permite un acceso más eficiente a los datos.

El formato BAM puede ordenarse según las coordenadas genómicas mediante **samtools sort** e indexarse con **samtools index**, lo que facilita el acceso aleatorio a regiones específicas del genoma sin necesidad de leer el archivo completo. Estas características hacen que el formato BAM sea el estándar utilizado por la mayoría de las herramientas de análisis posteriores, como **GATK**, **bcftools**, **VCFtools** y **IGV**, para el llamado de variantes, la visualización de alineamientos y otros análisis genómicos.

<p align="center">
  <img width="616" height="328" alt="image" src="https://github.com/user-attachments/assets/186a9bd1-b829-4b6b-aa8e-3715c41aa6ee" />




</p>

<p align="center">
<b>Figura 3.</b> Estructura general de un archivo SAM (Sequence Alignment/Map). El archivo está compuesto por una sección de cabecera (Header), que almacena información sobre el genoma de referencia y el proceso de alineamiento, y una sección de alineamientos (Alignment section), donde cada línea representa una lectura alineada al genoma. En cada registro se incluyen campos obligatorios como el nombre de la lectura (QNAME), el campo FLAG, el cromosoma de referencia (RNAME), la posición de alineamiento (POS), la calidad de mapeo (MAPQ), la cadena CIGAR, la secuencia (SEQ) y las calidades de las bases (QUAL), además de campos opcionales con información adicional del alineamiento. Adaptado de Li et al. (2009).
  
</p>

El archivo SAM está compuesto por una sección de cabecera (*Header*), que almacena información sobre el genoma de referencia y los parámetros del alineamiento, y una sección de alineamientos (*Alignment section*), donde cada línea representa una lectura alineada al genoma de referencia. Cada registro contiene información esencial para el análisis posterior, incluyendo el identificador de la lectura, la posición de alineamiento, la calidad del mapeo (**MAPQ**), la cadena **CIGAR**, la secuencia de nucleótidos y otros campos que describen las características del alineamiento. *Adaptado de Li et al. (2009).*

| Campo | Descripción |
|--------|-------------|
| **QNAME** | Identificador único de la lectura (*Query template name*). |
| **FLAG** | Campo numérico que codifica, mediante representación binaria, las características y el estado del alineamiento. |
| **RNAME** | Nombre de la secuencia de referencia (cromosoma o contig) donde se alineó la lectura. |
| **POS** | Posición inicial (base 1) del alineamiento sobre el genoma de referencia. |
| **MAPQ** | Calidad del alineamiento (*Mapping Quality Score*), que estima la confianza del mapeo. |
| **CIGAR** | Describe cómo se alineó la lectura respecto al genoma de referencia, indicando coincidencias, inserciones, deleciones y otras operaciones. |
| **RNEXT** | Nombre de la secuencia de referencia donde se alineó la lectura pareja (*paired-end*). |
| **PNEXT** | Posición inicial de la lectura pareja en el genoma de referencia. |
| **TLEN** | Longitud observada del fragmento de ADN secuenciado (*Template Length*). |
| **SEQ** | Secuencia de nucleótidos correspondiente a la lectura. |
| **QUAL** | Calidad Phred de cada nucleótido presente en la lectura. |

Una vez generado el archivo BAM, es indispensable evaluar la calidad del alineamiento antes del llamado de variantes y realizar un filtrado del mismo. Una mala calidad del alineamiento puede producir errores sistemáticos durante la identificación de SNPs e indels. Las herramientas más utilizadas para esta etapa son **Qualimap**, **SAMtools stats** e **IGV (Integrative Genomics Viewer)**.

Los principales parámetros evaluados incluyen:

- porcentaje de lecturas alineadas;
- profundidad de cobertura;
- cobertura del genoma;
- distribución del contenido GC;
- tasa de duplicados;
- tamaño del inserto;
- distribución del MAPQ;
- perfiles de clipping;
- contenido nucleotídico.

Estas métricas tambien pueden permitir detectar sesgos experimentales o problemas durante la preparación de bibliotecas y el proceso de secuenciación.

---

### Lecturas Single-End y Paired-End

Las tecnologías NGS permiten secuenciar un único extremo del fragmento (Single-End) o ambos extremos (Paired-End). En Single-End cada fragmento produce únicamente una lectura. Su principal ventaja consiste en un menor costo experimental. No obstante, proporciona menor información sobre la estructura del ADN. Con Paired-End cada fragmento genera dos lecturas. Ambas deben alinearse respetando: orientación; distancia esperada entre lecturas; orden correcto. La información adicional mejora considerablemente: la precisión del alineamiento; la resolución de regiones repetitivas; la detección de indels y la detección de variantes estructurales. Por esta razón, la mayoría de los estudios modernos de resecuenciación utilizan secuenciación paired-end.

---

## Alineamiento de lecturas largas

Las plataformas de secuenciación de tercera generación (Nanopore de Oxford Nanopore Technologies y PacBio de Pacific Biosciences) generan lecturas de longitudes muy superiores a Illumina —típicamente entre 10,000 y 100,000 pb, con registros de hasta 4 Mb en Nanopore, pero con tasas de error mucho más altas. Esto crea un problema fundamental para los alineadores basados en BWT:

El índice BWT funciona buscando coincidencias exactas o casi exactas entre la lectura y la referencia. Con tasas de error del 1-5%, una lectura de Nanopore de 10,000 pb puede tener entre hasta 500 errores distribuidos a lo largo de su longitud. Un alineador BWT intentaría dividir esa lectura en semillas exactas cada vez más cortas hasta encontrar coincidencias, lo que se vuelve computacionalmente prohibitivo: las semillas serían tan cortas que habría miles de posibles posiciones en el genoma donde podrían alinear, generando una explosión combinatoria.

La tecnología PacBio HiFi (también llamado CCS, Circular Consensus Sequencing) merece mención aparte porque genera lecturas largas (~15–20 kb) pero con tasas de error muy bajas (~0.1%), comparables a Illumina. Esto ocurre porque la polimerasa lee el mismo fragmento circular múltiples veces y genera una secuencia consenso. Por lo que BWA-MEM2 también funciona razonablemente bien en este caso, tambien existe pbmm2: el alineador oficial de PacBio, basado en la tecnología Minimap2 pero con optimizaciones específicas para HiFi.

En este contexto, en cuanto alineamiento de lecturas largas se trate Minimap2 y NGMLR se han establecido como uno los alineadores más empleados.

### Minimap2 y los minimizers 

Desarrollado por: Heng Li (2018), el mismo autor de BWA. Un minimizer es una forma inteligente de reducir la cantidad de k-mers que hay que comparar sin perder la capacidad de encontrar alineamientos. Funciona mediante una ventana deslizante de tamaño w que se mueve a lo largo de la secuencia. En cada posición de la ventana, hay w k-mers posibles. El minimizer es simplemente el k-mer lexicográficamente menor (o el de menor valor hash) dentro de esa ventana. Solo ese k-mer se guarda en el índice.

Por ejemplo, con w=5 y k=3, en la secuencia ACGTACGT:

Ventana 1: ACG, CGT, GTA, TAC, ACG → minimizer: ACG
Ventana 2: CGT, GTA, TAC, ACG, CGT → minimizer: ACG
Ventana 3: GTA, TAC, ACG, CGT, GTA → minimizer: ACG

Esto reduce drásticamente el número de k-mers que hay que indexar y comparar (típicamente por un factor de w/2), sin perder la capacidad de detectar regiones homólogas entre secuencias divergentes.

Como funciona: 

1. Indexación: Minimap2 construye un índice de minimizers del genoma de referencia mucho más compacto que un índice BWT completo.
2. Búsqueda de semillas (seeding): Para cada lectura larga, calcula sus minimizers y los busca en el índice de la referencia. Dado que las lecturas son muy largas, aunque tengan muchos errores, habrá suficientes minimizers compartidos con la referencia para identificar la región correcta.
3. Encadenamiento (chaining): Las semillas coincidentes se agrupan en cadenas colineares usando programación dinámica. Una cadena es un conjunto de semillas que aparecen en el mismo orden y orientación tanto en la lectura como en la referencia, lo que indica que provienen de la misma región genómica. Este paso es clave: permite manejar grandes gaps (intrones, inserciones, deleciones) que serían problemáticos para BWA.
4. Alineamiento local (alignment): Solo las regiones entre semillas de la misma cadena necesitan alineamiento detallado. Minimap2 usa el algoritmo de programación dinámica acelerada con instrucciones SIMD (similar al algoritmo de Smith-Waterman pero mucho más rápido) para rellenar los gaps entre semillas.
5. Salida en formato SAM/PAF: El resultado es un alineamiento con CIGAR string que describe exactamente las coincidencias, sustituciones, inserciones y deleciones.

Minimap2 tiene diferentes configuraciones según el tipo de dato: PresetUsomap-ontLecturas Nanopore (alta tasa de error, errores aleatorios)map-pbLecturas PacBio CLR (alta tasa de error, errores sistemáticos)map-hifiLecturas PacBio HiFi/CCS (baja tasa de error, ~0.1%)srLecturas cortas Illumina paired-endspliceAlineamiento de ARN-seq con splicingasm5Ensamblaje vs referencia muy similar (<5% divergencia)


<p align="center">
  <img width="708" height="662" alt="image" src="https://github.com/user-attachments/assets/2653697d-0a0c-4f74-839c-32c83826ef45" />


</p>

<p align="center">
<b>Figura 4.</b> Aplicación de minimizadores en la alineación de lecturas. Un alineador de lecturas típico que sigue el enfoque de alineación de cadena de semillas primero encuentra minimizadores de referencia y los almacena en una tabla hash. Las semillas son subcadenas (minimizadores) de la referencia o de la lectura. Las semillas que coinciden entre la lectura y la referencia se denominan anclas, que se encuentran consultando los minimizadores de lectura en la tabla hash. Luego, las anclas se encadenan y finalmente se alinean las bases.
  
</p>

### NGMLR

Minimap2 es excelente para alineamientos generales, pero tiene una limitación importante en regiones con variantes estructurales (SVs): grandes inserciones, deleciones, inversiones o translocaciones. En estas regiones, el encadenamiento de minimizers de Minimap2 puede generar alineamientos que "saltan" sobre la variante sin detectarla correctamente, o que dividen la lectura en múltiples alineamientos secundarios en lugar de un solo alineamiento con el gap correcto. NGMLR fue diseñado específicamente para maximizar la precisión del alineamiento en regiones con SVs.

Algoritmo de NGMLR:

1. Indexación con k-mers: NGMLR indexa la referencia con k-mers convex (convex k-mers), una variante que es más robusta a errores de secuenciación que los k-mers estándar. Busca k-mers de la lectura en la referencia para identificar regiones candidatas de alineamiento, similar a Minimap2 pero con parámetros más conservadores.

2. Alineamiento con modelo de gap convexo: La diferencia clave de NGMLR está en el modelo de penalización de gaps que usa durante el alineamiento: Los alineadores estándar usan una penalización afín de gaps: penalización de apertura + penalización por cada base del gap. Este modelo penaliza mucho las deleciones grandes, lo que hace que el alineador prefiera dividir la lectura en múltiples fragmentos antes que reportar una deleción de 10,000 pb. NGMLR usa una penalización convexa de gaps: la penalización crece rápidamente al inicio del gap pero se satura para gaps grandes. Matemáticamente: gap_penalty = a + b × log(gap_length). Esto hace que gaps grandes sean tan o más "baratos" que muchos gaps pequeños, lo que favorece el reporte correcto de grandes SVs en lugar de fragmentar el alineamiento.

---

# Un flujo de trabajo completo para mapear lecturas cortas contra un genoma de referencia. 

En este módulo se procesaron tres muestras de *Solanum sección lycopersicum* secuenciadas por WGS con Illumina: una accesión de *S. lycopersicum var. cerasiforme* (SLC; SRR31477438), una accesión de *S. lycopersicum var. lycopersicum* (LA1924; SRR38359005) y una accesión de *S. pimpinellifolium* (SRR37254991), mapeadas contra el genoma de referencia del tomate cultivado (*S. lycopersicum var. lycopersicum* Micro-Tom, ensamblaje SLM_r2.1 (GCF_036512215.1; 832.8 Mb). Las lecturas WGS de las muestras fueron descargadas desde la base de datos SRA del NCBI usando fasterq-dump, en formato FASTA.

---

Todos los análisis se ejecutaron en el clúster de cómputo HPC perteneciente a la Facultad de Ciencias de la Universidad Nacional de Colombia mediante scripts SLURM, utilizando el gestor de paquetes Conda y entornos virtuales asociados. La instalación y resolución de dependencias se ejecuto a través de Anaconda y Miniconda.

```

# Carga del módulo de conda
module load envs/anaconda3
source /scratchsan1/anaconda3/etc/profile.d/conda.sh

```
## 1. Indexación del genoma de referencia

Antes de realizar el alineamiento de las lecturas, el genoma de referencia debe indexarse. Estos archivos permiten que las herramientas de alineamiento y análisis accedan rápidamente a las secuencias del genoma sin tener que recorrer el archivo FASTA completo en cada operación.

Se generaron tres índices complementarios: Índice de BWA-MEM2 (bwa-mem2 index): genera un conjunto de archivos auxiliares basados en el algoritmo Burrows–Wheeler Transform (BWT) y el FM-index. Índice FASTA de SAMtools (samtools faidx): crea un archivo con extensión .fai que almacena la posición de inicio, longitud y coordenadas de cada cromosoma o secuencia del genoma de referencia. Utilizado por herramientas posteriores. Diccionario de secuencias de GATK (CreateSequenceDictionary): genera un archivo .dict que contiene los nombres, longitudes y metadatos de cada secuencia presente en el genoma de referencia. Este archivo es requerido por GATK para verificar la consistencia entre el genoma de referencia y los archivos BAM durante etapas posteriores como el llamado de variantes.

```

# Índice BWA-MEM2
bwa-mem2 index GCF_036512215.1_SLM_r2.1_genomic.fna

# Índice SAMtools (para acceso aleatorio)
samtools faidx GCF_036512215.1_SLM_r2.1_genomic.fna

# Diccionario de secuencias (requerido por GATK)
gatk CreateSequenceDictionary \
    -R GCF_036512215.1_SLM_r2.1_genomic.fna

```

## 2. Alineamiento de las lecturas al genoma de referencia

Las lecturas fueron alineadas utilizando BWA-MEM2. Se añadió un Read Group (RG) mediante el parámetro -R. El Read Group es un conjunto de metadatos que queda almacenado en el archivo BAM e identifica el origen de las lecturas. Incluye información como el nombre de la muestra (SM), un identificador del conjunto de datos (ID), la plataforma de secuenciación (PL, por ejemplo, ILLUMINA) y la biblioteca de secuenciación (LB). Estos datos permiten que GATK reconozca a qué muestra pertenece cada lectura y procesen correctamente los archivos durante el llamado de variantes y otros análisis posteriores.

Durante el alineamiento se incorporó un Read Group (RG) para cada muestra mediante el parámetro -R. Este campo contiene información sobre el identificador de la muestra (SM), el identificador del experimento (ID), la plataforma de secuenciación (PL) y la biblioteca utilizada (LB). La inclusión de esta información es indispensable para herramientas posteriores como GATK, ya que permite distinguir correctamente las muestras durante el llamado y filtrado de variantes.

El resultado generado por BWA-MEM2 se produjo inicialmente en formato SAM y fue enviado directamente mediante una tubería (pipe) a SAMtools sort, evitando la creación de archivos intermedios. SAMtools ordenó los alineamientos según sus coordenadas genómicas y almacenó el resultado en formato BAM, un formato binario comprimido que reduce el espacio de almacenamiento y permite un acceso mucho más eficiente durante las etapas posteriores del análisis.

El alineamiento se realizó empleando 32 hilos de procesamiento tanto para BWA-MEM2 como para SAMtools, aprovechando el paralelismo disponible en el servidor de cómputo

# Referencias

- Broad Institute. (2024). *Genome Analysis Toolkit (GATK) Best Practices*. https://gatk.broadinstitute.org/

- Danecek, P., Bonfield, J. K., Liddle, J., Marshall, J., Ohan, V., Pollard, M. O., ... Li, H. (2021). Twelve years of SAMtools and BCFtools. *GigaScience, 10*(2), giab008.

- Langmead, B., Trapnell, C., Pop, M., & Salzberg, S. L. (2009). Ultrafast and memory-efficient alignment of short DNA sequences to the human genome. *Genome Biology, 10*(3), R25.

- Langmead, B., & Salzberg, S. L. (2012). Fast gapped-read alignment with Bowtie 2. *Nature Methods, 9*(4), 357–359.

- Li, H., & Durbin, R. (2009). Fast and accurate short read alignment with Burrows–Wheeler Transform. *Bioinformatics, 25*(14), 1754–1760.

- Li, H. (2011). A statistical framework for SNP calling, mutation discovery and population genetic parameter estimation from sequencing data. *Bioinformatics, 27*(21), 2987–2993.

- McKenna, A., Hanna, M., Banks, E., Sivachenko, A., Cibulskis, K., Kernytsky, A., ... DePristo, M. A. (2010). The Genome Analysis Toolkit: A MapReduce framework for analyzing next-generation DNA sequencing data. *Genome Research, 20*(9), 1297–1303.

- Van der Auwera, G. A., & O'Connor, B. D. (2020). *Genomics in the Cloud*. O'Reilly Media.
