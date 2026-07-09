# NGS Data Analysis - Mapping and Variant Calling

> Una guía sobre los fundamentos del mapeo de lecturasm y la llamada de variantes y un flujo de trabajo completo con datos de secuenciación de nueva generación (Next Generation Sequencing, NGS)

# Introducción

El análisis de datos de secuenciación de nueva generación (Next Generation Sequencing, NGS) constituye uno de los pilares de la genómica moderna. El enorme volumen de información generado por estas tecnologías requiere algoritmos computacionales especializados capaces de transformar millones de lecturas cortas en información biológicamente interpretable. Entre las etapas más importantes de este proceso se encuentran el alineamiento de lecturas contra un genoma de referencia y el llamado de variantes. En un flujo de análisis típico de datos NGS, las lecturas obtenidas por el secuenciador son sometidas inicialmente a un control de calidad para eliminar adaptadores, secuencias contaminantes y bases de baja calidad. Posteriormente, las lecturas filtradas se alinean contra un genoma de referencia mediante algoritmos de mapeo, generando archivos de alineamiento que posteriormente sirven como entrada para herramientas de llamdo de variantes. Finalmente, las variantes detectadas son filtradas y anotadas antes de ser utilizadas en estudios de diversidad genética, genética de poblaciones, evolución o mejoramiento genético.

En este módulo se describen los fundamentos teóricos del alineamiento de lecturas y del llamado de variantes en datos de secuenciación de nueva generación (NGS). Se abordan los principios biológicos, computacionales y estadísticos que sustentan estas etapas y se presenta un flujo de trabajo completo para el análisis de datos NGS, que comprende la limpieza de lecturas, el alineamiento contra el genoma de referencia de Solanum lycopersicum, el llamado de variantes y el cálculo de estadísticas de diversidad genética a partir de datos de Solanum pimpinellifolium, Solanum lycopersicum var. cerasiforme y Solanum lycopersicum var. lycopersicum.

---

# 4.1 Introducción al análisis de datos de secuenciación de nueva generación (NGS)

Las tecnologías de secuenciación de nueva generación (NGS) revolucionaron la biología molecular al permitir la obtención simultánea de millones de fragmentos de ADN en una única corrida de secuenciación. A diferencia del método clásico de Sanger, que produce una única secuencia por reacción, las plataformas NGS generan millones o incluso miles de millones de lecturas (reads), reduciendo considerablemente el costo por base secuenciada y aumentando la velocidad de adquisición de datos.

Actualmente, plataformas como Illumina, Oxford Nanopore Technologies y Pacific Biosciences permiten abordar preguntas biológicas relacionadas con la identificación de variantes genómicas, ensamblaje de genomas, análisis transcriptómicos, estudios metagenómicos y caracterización epigenética.

Sin embargo, el resultado directo del proceso de secuenciación no corresponde al genoma completo del organismo, sino a millones de pequeñas secuencias independientes denominadas **lecturas** (*reads*), cuya longitud puede variar desde aproximadamente 50 hasta varios miles de nucleótidos dependiendo de la tecnología utilizada.

Estas lecturas representan únicamente fragmentos del ADN original, por lo que deben ser procesadas computacionalmente para reconstruir la información biológica del genoma. El análisis bioinformático transforma estos datos crudos en información útil mediante una serie de etapas secuenciales que incluyen:

1. Control de calidad de las lecturas.
2. Eliminación de adaptadores y secuencias de baja calidad.
3. Alineamiento contra un genoma de referencia.
4. Procesamiento de los alineamientos.
5. Llamado de variantes.
6. Filtrado de variantes.
7. Anotación funcional.

Cada una de estas etapas reduce progresivamente la incertidumbre asociada a los datos experimentales, permitiendo identificar con mayor precisión las diferencias genéticas reales entre individuos.

---

# 4.2 Alineamiento de secuencias

## 4.2.1 Concepto

El alineamiento de secuencias constituye uno de los problemas fundamentales de la bioinformática. Consiste en establecer la correspondencia óptima entre dos o más secuencias de ADN, ARN o proteínas con el propósito de identificar regiones conservadas y diferencias evolutivas.

Desde el punto de vista computacional, el alineamiento busca maximizar la similitud entre secuencias permitiendo la aparición de sustituciones, inserciones y deleciones cuando estas representan mejor la historia evolutiva de las moléculas comparadas.

En experimentos de NGS, el alineamiento recibe el nombre específico de **read mapping**, ya que el objetivo consiste en localizar la posición más probable del genoma donde se originó cada lectura generada por el secuenciador.

La correcta ubicación de las lecturas resulta indispensable para todas las etapas posteriores del análisis. Si una lectura es alineada en una posición incorrecta, cualquier diferencia observada respecto al genoma de referencia podría interpretarse erróneamente como una variante genética cuando en realidad corresponde a un error de alineamiento.

Por esta razón, el alineamiento constituye el paso crítico sobre el cual depende la precisión del llamado de variantes.

---

## 4.2.2 Importancia biológica

El alineamiento de secuencias posee aplicaciones en prácticamente todas las áreas de la biología molecular.

Entre sus principales aplicaciones destacan:

- identificación de SNPs e indels;
- estudios filogenéticos;
- genética de poblaciones;
- análisis de expresión génica mediante RNA-Seq;
- identificación de mutaciones asociadas a enfermedades;
- estudios de adaptación evolutiva;
- análisis de diversidad genética;
- programas de mejoramiento vegetal y animal.

En proyectos de resecuenciación genómica, el alineamiento permite comparar directamente el ADN de un individuo con un genoma de referencia para detectar diferencias nucleotídicas responsables de variaciones fenotípicas.

---

## 4.2.3 Desafíos computacionales

El alineamiento de millones de lecturas representa un problema computacional extremadamente complejo.

Un experimento típico de secuenciación puede generar más de 500 millones de lecturas. Comparar cada una de ellas contra todas las posiciones posibles de un genoma de aproximadamente 900 Mb, como el de *Solanum lycopersicum*, requeriría un tiempo computacional prohibitivo si se utilizaran algoritmos clásicos de comparación exhaustiva.

Además, los algoritmos deben considerar simultáneamente múltiples fuentes de variación, entre ellas:

- errores de secuenciación;
- mutaciones reales;
- inserciones;
- deleciones;
- regiones repetitivas;
- duplicaciones génicas;
- baja calidad de bases.

Por ello, los alineadores modernos emplean estructuras de datos comprimidas y algoritmos heurísticos que permiten acelerar considerablemente el proceso sin comprometer significativamente la precisión.

---

# 4.3 Alineamiento de lecturas contra un genoma de referencia

El alineamiento contra referencia consiste en asignar cada lectura de ADN a la región del genoma donde probablemente fue originada.

Durante este proceso, el alineador compara la secuencia de cada lectura con millones de posiciones posibles del genoma y calcula cuál de ellas representa el alineamiento más probable.

La calidad de este proceso depende de diversos factores, incluyendo:

- longitud de la lectura;
- calidad Phred;
- presencia de variantes reales;
- porcentaje de regiones repetitivas;
- tamaño del genoma;
- calidad del ensamblaje de referencia.

Cuando existe un genoma de referencia de alta calidad, el alineamiento permite obtener una representación precisa de las diferencias genéticas entre la muestra y dicho genoma.

No obstante, algunas regiones presentan secuencias altamente repetitivas o duplicadas, dificultando la asignación única de determinadas lecturas. En estos casos, los algoritmos asignan una probabilidad de confianza denominada **Mapping Quality (MAPQ)**, la cual refleja la certeza estadística de la posición seleccionada.

Un alineamiento de alta calidad constituye el requisito indispensable para el descubrimiento confiable de variantes.

---

# 4.4 Indexación del genoma de referencia

El principal desafío computacional del alineamiento consiste en realizar búsquedas extremadamente rápidas dentro de genomas que contienen cientos o miles de millones de nucleótidos.

Para resolver este problema, los alineadores modernos no comparan directamente cada lectura contra toda la secuencia del genoma. En su lugar, construyen previamente un índice comprimido que facilita las búsquedas.

## 4.4.1 Transformada de Burrows-Wheeler (BWT)

La Transformada de Burrows-Wheeler (Burrows-Wheeler Transform, BWT) constituye una transformación reversible que reorganiza los caracteres del genoma para agrupar regiones similares sin modificar la información original.

Aunque inicialmente fue desarrollada para algoritmos de compresión de datos, posteriormente se demostró que esta transformación permite realizar búsquedas extremadamente eficientes sobre secuencias biológicas.

Su principal ventaja consiste en reducir significativamente el espacio de memoria requerido para almacenar el genoma, manteniendo al mismo tiempo tiempos de búsqueda muy bajos.

## 4.4.2 FM-index

Sobre la BWT se construye el denominado **FM-index**, una estructura de datos que permite localizar rápidamente cualquier subsecuencia dentro del genoma sin necesidad de recorrerlo completamente.

Gracias al FM-index, herramientas como Bowtie2 y BWA pueden localizar millones de lecturas utilizando únicamente unos pocos gigabytes de memoria RAM, incluso para genomas de gran tamaño.

La combinación BWT–FM-index constituye actualmente el estándar para el alineamiento de lecturas cortas debido a su excelente equilibrio entre velocidad, consumo de memoria y precisión.

---

# 4.5 Algoritmos modernos de alineamiento

Los alineadores actuales emplean una estrategia denominada **Seed-and-Extend**, diseñada para acelerar el proceso de búsqueda.

En lugar de comparar la lectura completa contra todas las posiciones del genoma, el algoritmo divide inicialmente cada lectura en pequeños fragmentos llamados **semillas** (*seeds*), generalmente de entre 20 y 30 nucleótidos.

Cada una de estas semillas se busca rápidamente utilizando el FM-index. Una vez encontrada una coincidencia potencial, el algoritmo extiende el alineamiento hacia ambos extremos utilizando técnicas de programación dinámica.

Para esta fase de extensión se utilizan variantes optimizadas del algoritmo de Smith-Waterman, el cual permite identificar el alineamiento local de mayor puntuación considerando coincidencias, sustituciones, inserciones y deleciones.

Este enfoque presenta múltiples ventajas:

- disminuye considerablemente el tiempo de ejecución;
- reduce el consumo de memoria;
- permite detectar pequeñas inserciones y deleciones;
- mejora la precisión del alineamiento en regiones variables;
- facilita el procesamiento de millones de lecturas en tiempos razonables.

La estrategia Seed-and-Extend representa actualmente la base computacional de la mayoría de los alineadores modernos utilizados en genómica, incluyendo Bowtie2, BWA-MEM y otros algoritmos ampliamente empleados en proyectos de resecuenciación.
# 4.6 Bowtie y Bowtie2

## 4.6.1 Introducción

El alineamiento de millones de lecturas generadas por tecnologías de secuenciación masiva constituye uno de los problemas computacionales más importantes de la bioinformática moderna. Debido al enorme volumen de datos generado por plataformas NGS, resulta inviable comparar exhaustivamente cada lectura contra todas las posiciones posibles del genoma mediante algoritmos clásicos de alineamiento.

Con el propósito de resolver este problema, Langmead et al. desarrollaron **Bowtie** en 2009, uno de los primeros alineadores ultrarrápidos para lecturas cortas basado en la Transformada de Burrows-Wheeler (Burrows-Wheeler Transform, BWT) y el FM-index. Posteriormente, Langmead y Salzberg publicaron **Bowtie2**, una versión completamente rediseñada capaz de detectar inserciones y deleciones, realizar alineamientos locales y soportar lecturas paired-end, convirtiéndose en uno de los alineadores más utilizados para datos Illumina.

Bowtie2 es actualmente empleado en proyectos de resecuenciación genómica, secuenciación dirigida, metagenómica y numerosos estudios de genética de poblaciones debido a su equilibrio entre velocidad, precisión y bajo consumo de memoria.

---

## 4.6.2 Principio de funcionamiento

El funcionamiento interno de Bowtie2 puede dividirse en cinco etapas principales.

### 1. Construcción del índice

Antes de iniciar el alineamiento, el genoma de referencia es transformado mediante la Burrows-Wheeler Transform y posteriormente indexado utilizando el FM-index.

Este índice permite localizar rápidamente cualquier subsecuencia dentro del genoma sin recorrer completamente la secuencia de referencia.

Esta etapa solamente debe ejecutarse una vez para cada genoma.

---

### 2. Selección de semillas

Cada lectura es dividida en pequeños fragmentos (seeds).

Las semillas corresponden a pequeños kmers cuya longitud suele variar entre 20 y 30 nucleótidos.

Estas semillas son buscadas dentro del índice del genoma.

---

### 3. Búsqueda rápida

Cada seed es localizada mediante el FM-index.

En esta etapa el algoritmo identifica únicamente regiones candidatas donde podría encontrarse la lectura completa.

Esto reduce drásticamente el número de comparaciones necesarias.

---

### 4. Extensión del alineamiento

Una vez encontrada una coincidencia potencial, Bowtie2 utiliza programación dinámica basada en Smith-Waterman para extender el alineamiento hacia ambos extremos.

Durante esta fase se permiten:

- sustituciones
- inserciones
- deleciones
- regiones parcialmente alineadas

Por esta razón Bowtie2 puede detectar pequeños indels con mucha mayor precisión que Bowtie original.

---

### 5. Evaluación del alineamiento

Finalmente el algoritmo calcula un puntaje para cada alineamiento considerando:

- número de mismatches;
- inserciones;
- deleciones;
- calidad de las bases;
- alineamientos alternativos.

A partir de esta información se calcula posteriormente el **Mapping Quality Score (MAPQ)**.

---

# 4.6.3 Modos de alineamiento

Bowtie2 permite dos estrategias principales.

## End-to-End Alignment

En este modo toda la lectura debe alinearse completamente contra el genoma.

No se permite eliminar nucleótidos en los extremos.

Este método resulta apropiado cuando las lecturas poseen excelente calidad y prácticamente no contienen adaptadores.

Su ventaja principal consiste en producir alineamientos más estrictos.

Sin embargo, puede disminuir la tasa de mapeo cuando existen regiones de baja calidad.

---

## Local Alignment

En el alineamiento local Bowtie2 permite recortar automáticamente regiones terminales de baja calidad mediante **soft clipping**.

El algoritmo conserva únicamente la parte de la lectura que produce el mejor alineamiento.

Este modo resulta especialmente útil cuando:

- permanecen adaptadores residuales;
- existen errores de secuenciación en los extremos;
- algunas bases presentan baja calidad.

Actualmente constituye uno de los modos más utilizados para datos Illumina.

---

# 4.6.4 Lecturas Single-End y Paired-End

Las tecnologías NGS permiten secuenciar un único extremo del fragmento (Single-End) o ambos extremos (Paired-End).

## Single-End

Cada fragmento produce únicamente una lectura.

Su principal ventaja consiste en un menor costo experimental.

No obstante, proporciona menor información sobre la estructura del ADN.

---

## Paired-End

En este caso cada fragmento genera dos lecturas.

Ambas deben alinearse respetando:

- orientación;
- distancia esperada entre lecturas;
- orden correcto.

La información adicional mejora considerablemente:

- la precisión del alineamiento;
- la resolución de regiones repetitivas;
- la detección de indels;
- la detección de variantes estructurales.

Por esta razón, la mayoría de los estudios modernos de resecuenciación utilizan secuenciación paired-end.

---

# 4.6.5 Ventajas de Bowtie2

Entre sus principales ventajas destacan:

- muy bajo consumo de memoria;
- elevada velocidad de alineamiento;
- excelente precisión para lecturas cortas;
- soporte para indels;
- alineamiento local y global;
- soporte para paired-end;
- amplia compatibilidad con SAMtools y GATK;
- integración en prácticamente todos los pipelines de bioinformática.

---

# 4.6.6 Limitaciones

A pesar de sus ventajas, Bowtie2 presenta algunas limitaciones.

No fue diseñado para:

- lecturas extremadamente largas (Oxford Nanopore o PacBio);
- alineamientos altamente divergentes;
- RNA-Seq con intrones complejos.

En estos casos suelen emplearse herramientas como STAR, Minimap2 o BWA-MEM.

---

# 4.6.7 Comparación con otros alineadores

| Herramienta | Tipo principal | Indels | Local | Spliced | Lecturas largas |
|-------------|---------------|---------|--------|-----------|----------------|
| Bowtie | Lecturas cortas | Limitado | No | No | No |
| Bowtie2 | Lecturas cortas | Sí | Sí | No | Limitado |
| BWA-MEM | ADN genómico | Sí | Sí | No | Sí |
| STAR | RNA-Seq | Sí | Sí | Sí | No |
| Minimap2 | Long Reads | Sí | Sí | Sí | Sí |

---

# 4.7 Formatos SAM y BAM

Una vez finalizado el alineamiento, los resultados deben almacenarse en un formato estándar que permita ser interpretado por diferentes programas bioinformáticos.

Con este propósito se desarrolló el formato **SAM (Sequence Alignment Map)**.

Debido a que los archivos SAM pueden ocupar decenas o cientos de gigabytes, normalmente se convierten a su versión binaria comprimida denominada **BAM (Binary Alignment Map)**.

Estos formatos constituyen actualmente el estándar universal para representar alineamientos de secuencias.

---

## 4.7.1 Estructura general del archivo SAM

Un archivo SAM posee dos partes.

### Cabecera (Header)

Comienza con el símbolo **@**.

Contiene información sobre:

- versión del formato;
- nombre del genoma;
- longitud de los cromosomas;
- programa utilizado para el alineamiento.

---

### Registros

Cada línea representa una lectura individual alineada contra el genoma.

Cada registro contiene once campos obligatorios y diversos campos opcionales.

Los principales son:

| Campo | Descripción |
|--------|-------------|
| QNAME | Nombre de la lectura |
| FLAG | Estado del alineamiento |
| RNAME | Cromosoma |
| POS | Posición inicial |
| MAPQ | Calidad del alineamiento |
| CIGAR | Descripción del alineamiento |
| RNEXT | Cromosoma de la lectura pareada |
| PNEXT | Posición de la pareja |
| TLEN | Tamaño del inserto |
| SEQ | Secuencia |
| QUAL | Calidad Phred |

---

# 4.7.2 Mapping Quality (MAPQ)

MAPQ representa la confianza estadística de que una lectura fue alineada en la posición correcta.

Mientras mayor sea el valor de MAPQ, menor es la probabilidad de que exista otro alineamiento igualmente probable.

Generalmente:

| MAPQ | Interpretación |
|------|----------------|
| 0 | Múltiples posiciones posibles |
| 10-20 | Baja confianza |
| 20-30 | Confianza moderada |
| >30 | Alta confianza |
| 60 | Alineamiento prácticamente único |

Durante el llamado de variantes suelen descartarse lecturas con valores bajos de MAPQ para reducir falsos positivos.

---

# 4.7.3 Cadena CIGAR

La cadena **CIGAR (Compact Idiosyncratic Gapped Alignment Report)** describe exactamente cómo fue alineada cada lectura respecto al genoma.

Las operaciones más comunes son:

| Símbolo | Significado |
|----------|-------------|
| M | Match o mismatch |
| I | Inserción |
| D | Deleción |
| N | Región omitida |
| S | Soft clipping |
| H | Hard clipping |
| = | Coincidencia exacta |
| X | Mismatch |

Ejemplo:

```
76M
```

La lectura coincide completamente.

Ejemplo:

```
35M2I39M
```

Existe una inserción de dos nucleótidos.

Ejemplo:

```
20S56M
```

Los primeros veinte nucleótidos fueron recortados mediante soft clipping.

---

# 4.7.4 FLAGS

El campo FLAG corresponde a un número entero que codifica múltiples propiedades del alineamiento mediante representación binaria.

Entre las más importantes se encuentran:

- lectura paired-end;
- lectura correctamente apareada;
- lectura no alineada;
- pareja no alineada;
- lectura en hebra reversa;
- duplicado por PCR;
- alineamiento secundario;
- alineamiento suplementario.

Estos indicadores permiten filtrar lecturas antes del llamado de variantes.

---

# 4.7.5 Conversión SAM → BAM

Debido al gran tamaño de los archivos SAM, normalmente se convierten inmediatamente al formato BAM utilizando SAMtools.

Posteriormente se realizan tres procedimientos esenciales:

1. Conversión a BAM.
2. Ordenamiento por coordenadas genómicas.
3. Indexación mediante archivos BAI.

La indexación permite acceder rápidamente a regiones específicas del genoma sin necesidad de cargar el archivo completo en memoria.

---

# 4.8 Evaluación de la calidad del alineamiento

Una vez generado el archivo BAM, resulta indispensable evaluar la calidad del alineamiento antes del llamado de variantes.

Una mala calidad del alineamiento puede producir errores sistemáticos durante la identificación de SNPs e indels.

Las herramientas más utilizadas para esta etapa son **Qualimap**, **SAMtools stats** e **IGV (Integrative Genomics Viewer)**.

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

Estas métricas permiten detectar sesgos experimentales o problemas durante la preparación de bibliotecas y el proceso de secuenciación.

---

# 4.9 Introducción al llamado de variantes

El objetivo del llamado de variantes consiste en identificar todas las posiciones del genoma donde las muestras presentan diferencias respecto al genoma de referencia.

Estas diferencias reciben el nombre de **variantes genéticas**.

Las variantes más comunes son:

## SNP

Cambio de un único nucleótido.

Ejemplo:

```
Referencia : A
Muestra    : G
```

---

## INDEL

Inserción o deleción de uno o varios nucleótidos.

Ejemplo:

```
Referencia : ATCGTT
Muestra    : ATGTT
```

---

## MNP

Sustitución simultánea de varios nucleótidos consecutivos.

---

## Variantes estructurales

Incluyen:

- inversiones;
- translocaciones;
- duplicaciones;
- grandes inserciones;
- grandes deleciones.

Generalmente requieren algoritmos especializados distintos de los llamadores convencionales de SNPs.

---

# 4.10 Concepto de genotipo

El llamado de variantes no solamente identifica la presencia de una mutación.

También determina el **genotipo** del individuo.

En organismos diploides existen tres posibilidades básicas.

| Genotipo | Significado |
|-----------|-------------|
| 0/0 | Homocigoto referencia |
| 0/1 | Heterocigoto |
| 1/1 | Homocigoto alternativo |

La determinación correcta del genotipo depende de múltiples factores:

- profundidad de cobertura;
- calidad Phred;
- calidad del alineamiento;
- distribución de alelos;
- probabilidad estadística.

Por ello, los llamadores modernos emplean modelos probabilísticos complejos en lugar de simples conteos de nucleótidos observados.
# 4.11 Errores asociados al llamado de variantes

El llamado de variantes constituye un proceso de inferencia estadística cuyo objetivo es distinguir diferencias genéticas reales de los errores introducidos durante la obtención y procesamiento de los datos de secuenciación. Aunque las plataformas modernas de NGS presentan tasas de error relativamente bajas, ningún experimento de secuenciación está completamente libre de errores. En consecuencia, no toda discrepancia observada entre una lectura y el genoma de referencia corresponde necesariamente a una variante biológica.

Las principales fuentes de error pueden clasificarse en errores experimentales, errores computacionales y errores derivados del propio genoma de referencia. El reconocimiento y control de estos factores constituye uno de los aspectos más importantes del análisis bioinformático.

## 4.11.1 Errores de secuenciación

Durante la síntesis y detección de nucleótidos pueden producirse errores en la identificación de las bases. Estos errores aumentan generalmente hacia los extremos de las lecturas debido a la disminución progresiva de la intensidad de fluorescencia y al incremento del ruido instrumental.

Cada base recibe una puntuación de calidad Phred que representa la probabilidad de que dicha base haya sido identificada incorrectamente. Una baja calidad de secuenciación incrementa la probabilidad de detectar falsos SNPs o falsos indels.

---

## 4.11.2 Errores durante la preparación de bibliotecas

La amplificación mediante PCR puede introducir mutaciones artificiales que posteriormente son interpretadas como variantes reales. Además, la sobreamplificación puede originar múltiples copias idénticas del mismo fragmento de ADN, conocidas como duplicados de PCR.

Estos duplicados generan una representación artificialmente elevada de determinados alelos y pueden sesgar la estimación del genotipo.

---

## 4.11.3 Errores de alineamiento

Los errores de mapeo constituyen una de las principales causas de falsos positivos durante el llamado de variantes.

Este problema ocurre principalmente cuando una lectura puede alinearse con alta similitud en múltiples regiones del genoma, situación frecuente en secuencias repetitivas, familias multigénicas o regiones altamente conservadas.

Los alineadores asignan un valor de Mapping Quality (MAPQ) para estimar la probabilidad de que una lectura haya sido ubicada correctamente.

---

## 4.11.4 Errores del genoma de referencia

El genoma de referencia representa únicamente un individuo o ensamblaje específico de la especie y no necesariamente refleja toda la diversidad genética existente.

Errores de ensamblaje, regiones faltantes, inversiones o secuencias incorrectamente ensambladas pueden provocar discrepancias sistemáticas entre las lecturas y la referencia.

---

## 4.11.5 Cobertura insuficiente

La profundidad de secuenciación determina el número de lecturas que cubren una determinada posición del genoma.

Una cobertura baja disminuye considerablemente la confianza en la estimación del genotipo debido a que un pequeño número de errores experimentales puede confundirse con variantes verdaderas.

Por el contrario, coberturas excesivamente altas pueden indicar la presencia de duplicados de PCR o regiones repetitivas.

---

# 4.12 Fundamentos estadísticos del llamado de variantes

Los primeros algoritmos de llamado de variantes utilizaban reglas simples basadas en el conteo de nucleótidos observados en cada posición. Sin embargo, este enfoque resultó insuficiente para analizar regiones con baja cobertura, errores de secuenciación o variantes complejas.

Actualmente, prácticamente todos los llamadores modernos emplean modelos probabilísticos basados en inferencia bayesiana.

Estos modelos integran simultáneamente la información proveniente de:

- calidad de las bases;
- calidad del alineamiento;
- profundidad de cobertura;
- frecuencia esperada de variantes;
- distribución de alelos.

El resultado final corresponde al genotipo con mayor probabilidad posterior.

---

# 4.12.1 Calidad Phred

La calidad de cada nucleótido se expresa mediante la escala Phred.

Esta escala representa logarítmicamente la probabilidad de error durante la identificación de una base.

Su definición matemática es:

\[
Q=-10\log_{10}(P_{error})
\]

donde:

- **Q** corresponde a la calidad Phred.
- **Perror** representa la probabilidad de error.

Por ejemplo:

| Calidad | Probabilidad de error |
|---------|-----------------------|
| Q20 | 1 % |
| Q30 | 0.1 % |
| Q40 | 0.01 % |

Debido a esta relación logarítmica, pequeñas diferencias en la puntuación Phred representan cambios importantes en la confiabilidad de las bases.

---

# 4.12.2 Genotype Likelihood

El primer paso del llamado de variantes consiste en calcular la probabilidad de observar las lecturas obtenidas suponiendo que el individuo posee un determinado genotipo.

Esta probabilidad recibe el nombre de **Genotype Likelihood**.

Por ejemplo, para un organismo diploide se evalúan generalmente tres posibilidades:

- Homocigoto referencia (AA)
- Heterocigoto (AB)
- Homocigoto alternativo (BB)

Cada lectura aporta evidencia a favor o en contra de cada uno de estos genotipos.

La probabilidad conjunta de todas las lecturas permite estimar cuál es el genotipo más compatible con los datos experimentales.

---

# 4.12.3 Inferencia bayesiana

La inferencia bayesiana constituye el fundamento estadístico de herramientas como GATK, FreeBayes y bcftools.

El objetivo consiste en calcular la probabilidad del genotipo considerando toda la evidencia disponible.

El modelo general puede expresarse como:

\[
P(Genotipo|Datos)\propto P(Datos|Genotipo)\times P(Genotipo)
\]

donde:

- **P(Datos|Genotipo)** corresponde a la verosimilitud (Likelihood).
- **P(Genotipo)** representa la probabilidad previa (Prior).
- **P(Genotipo|Datos)** corresponde a la probabilidad posterior.

El genotipo seleccionado será aquel con mayor probabilidad posterior.

---

# 4.12.4 Priors

Los priors representan el conocimiento previo disponible antes de analizar las lecturas.

Dependiendo del algoritmo pueden construirse utilizando:

- tasa esperada de heterocigosidad;
- frecuencia conocida de SNPs;
- bases de datos como dbSNP;
- equilibrio de Hardy-Weinberg;
- información proveniente de múltiples individuos.

La utilización de priors mejora considerablemente la precisión del llamado de variantes, especialmente en regiones con baja cobertura.

---

# 4.12.5 Inferencia en múltiples muestras

Cuando varias muestras son analizadas simultáneamente, la estimación de los genotipos incorpora información poblacional.

El supuesto más utilizado corresponde al equilibrio de Hardy-Weinberg, según el cual, si la frecuencia del alelo A es **p** y la del alelo B es **q**, las frecuencias esperadas de los genotipos son:

\[
P(AA)=p^2
\]

\[
P(AB)=2pq
\]

\[
P(BB)=q^2
\]

Este enfoque incrementa la sensibilidad para detectar variantes raras presentes únicamente en algunos individuos de la población.

---

# 4.13 Genome Analysis Toolkit (GATK)

El **Genome Analysis Toolkit (GATK)** es un conjunto de herramientas bioinformáticas desarrollado por el Broad Institute para el descubrimiento y genotipado de variantes genómicas.

Desde su publicación en 2010, GATK se ha convertido en uno de los estándares internacionales para el análisis de datos de resecuenciación debido a su elevada precisión y a la incorporación de modelos estadísticos avanzados.

Actualmente es ampliamente utilizado en proyectos de secuenciación del genoma completo (WGS), exomas, paneles dirigidos y estudios de genética de poblaciones.

---

# 4.13.1 Flujo Best Practices

El flujo de trabajo recomendado por GATK se organiza en cuatro etapas principales:

1. Preprocesamiento de los alineamientos.
2. Descubrimiento de variantes.
3. Genotipado conjunto.
4. Filtrado de variantes.

Cada una de estas etapas busca reducir progresivamente la incertidumbre de los datos hasta obtener un conjunto de variantes de alta confianza.

---

# 4.14 Preprocesamiento del archivo BAM

Antes del llamado de variantes es necesario optimizar los alineamientos mediante varias etapas de procesamiento.

## Ordenamiento

Los alineamientos se ordenan según su posición cromosómica utilizando **samtools sort**.

Este procedimiento facilita el acceso eficiente a regiones específicas del genoma.

---

## Indexación

Posteriormente se genera un índice (.bai) mediante **samtools index**, permitiendo el acceso aleatorio a cualquier posición del archivo BAM.

---

## Marcado de duplicados

Durante la amplificación por PCR pueden generarse múltiples copias del mismo fragmento de ADN.

La herramienta **MarkDuplicates**, incluida en Picard, identifica estas copias utilizando la posición de inicio y orientación de las lecturas.

Los duplicados no se eliminan físicamente, sino que son marcados para que puedan ser ignorados durante el llamado de variantes.

---

## Recalibración de calidad de bases (BQSR)

Los secuenciadores presentan errores sistemáticos asociados al ciclo de secuenciación, contexto nucleotídico y química utilizada.

La herramienta **BaseRecalibrator** estima dichos errores utilizando variantes conocidas y genera un modelo estadístico que posteriormente corrige las puntuaciones Phred mediante **ApplyBQSR**.

Este procedimiento mejora considerablemente la precisión del llamado de variantes.

---

# 4.15 HaplotypeCaller

HaplotypeCaller constituye el algoritmo central del pipeline moderno de GATK.

A diferencia de los llamadores clásicos, que evalúan cada posición del genoma de forma independiente, HaplotypeCaller analiza regiones completas del genoma reconstruyendo los posibles haplotipos presentes en cada individuo.

Este enfoque permite detectar con mayor precisión regiones que contienen múltiples SNPs e inserciones o deleciones cercanas.

## 4.15.1 Regiones activas

Inicialmente, HaplotypeCaller identifica regiones del genoma donde existe evidencia de variación.

Estas regiones reciben el nombre de **Active Regions**.

Únicamente estas regiones son sometidas al análisis intensivo, reduciendo considerablemente el tiempo computacional.

---

## 4.15.2 Reensamblaje local

Cada región activa es reensamblada localmente utilizando grafos de De Bruijn.

Este procedimiento reconstruye los posibles haplotipos presentes sin depender completamente del alineamiento original.

El reensamblaje local incrementa notablemente la precisión en regiones con múltiples indels.

---

## 4.15.3 Pair Hidden Markov Model (PairHMM)

Posteriormente, cada lectura es comparada contra todos los haplotipos candidatos mediante un modelo probabilístico denominado **Pair Hidden Markov Model (PairHMM)**.

Este algoritmo calcula la probabilidad de que cada lectura provenga de cada haplotipo considerando:

- errores de secuenciación;
- inserciones;
- deleciones;
- mismatches.

Las probabilidades obtenidas sirven posteriormente para calcular la verosimilitud de cada genotipo.

---

## 4.15.4 Generación de archivos GVCF

En lugar de producir directamente un archivo VCF tradicional, HaplotypeCaller puede ejecutarse en modo **GVCF (Genomic Variant Call Format)**.

El archivo GVCF almacena información tanto de posiciones variantes como no variantes, permitiendo posteriormente combinar múltiples individuos sin necesidad de repetir el llamado de variantes.

Este enfoque constituye la base del genotipado conjunto utilizado por GATK.

# 4.16 Genotipado conjunto (Joint Genotyping)

En estudios genómicos modernos es frecuente analizar simultáneamente decenas, cientos o incluso miles de individuos. Si cada muestra fuera analizada de manera independiente, una variante presente únicamente en pocos individuos podría no ser detectada debido a la baja evidencia estadística disponible en cada muestra por separado. Para superar esta limitación, GATK implementa una estrategia denominada **Joint Genotyping** o **genotipado conjunto**, mediante la cual todas las muestras son evaluadas simultáneamente utilizando un único modelo probabilístico.

Este enfoque incrementa la sensibilidad para detectar variantes raras, mejora la consistencia de los genotipos entre individuos y permite diferenciar con mayor precisión entre posiciones verdaderamente invariantes y regiones con cobertura insuficiente.

---

## 4.16.1 Formato GVCF

El primer paso del genotipado conjunto consiste en ejecutar **HaplotypeCaller** en modo **GVCF (Genomic Variant Call Format)**.

A diferencia del formato VCF tradicional, un archivo GVCF almacena información tanto de posiciones variantes como de regiones donde no se detectaron variantes, indicando el nivel de confianza asociado a cada sitio del genoma.

Esta estrategia ofrece varias ventajas:

- evita repetir el llamado de variantes cuando se incorporan nuevas muestras;
- permite analizar cohortes de gran tamaño;
- mejora la estimación de genotipos poco frecuentes;
- facilita el procesamiento distribuido en sistemas de alto rendimiento.

Cada individuo genera inicialmente un archivo GVCF independiente que posteriormente será integrado con el resto de las muestras.

---

## 4.16.2 GenomicsDBImport y CombineGVCFs

Una vez obtenidos todos los archivos GVCF, GATK proporciona dos herramientas para consolidar la información.

### GenomicsDBImport

Es el método recomendado para estudios con un gran número de muestras. Esta herramienta organiza la información de todos los GVCF en una base de datos optimizada para consultas rápidas y procesamiento paralelo.

Sus principales ventajas son:

- menor consumo de memoria;
- mayor velocidad;
- escalabilidad para cientos o miles de individuos.

### CombineGVCFs

Esta herramienta combina varios archivos GVCF en un único archivo. Aunque resulta adecuada para proyectos pequeños, presenta limitaciones de rendimiento cuando el número de muestras aumenta considerablemente.

---

## 4.16.3 GenotypeGVCFs

Una vez consolidados los archivos GVCF, la herramienta **GenotypeGVCFs** realiza el genotipado conjunto.

Durante este procedimiento, GATK:

1. analiza simultáneamente todas las muestras;
2. estima la frecuencia de cada alelo;
3. calcula nuevamente las probabilidades de cada genotipo;
4. genera un único archivo VCF multimuesta.

El análisis conjunto permite aumentar la potencia estadística del estudio y reducir la probabilidad de clasificar incorrectamente una variante como ausencia de variación debido a una cobertura insuficiente.

---

# 4.17 Filtrado de variantes

El llamado inicial de variantes está diseñado para maximizar la sensibilidad del análisis. En consecuencia, el conjunto de variantes obtenido suele contener tanto variantes verdaderas como falsos positivos derivados de errores de secuenciación, alineamiento o preparación de bibliotecas.

Por esta razón, el filtrado constituye una etapa indispensable del análisis bioinformático.

El objetivo consiste en eliminar variantes poco confiables conservando únicamente aquellas respaldadas por suficiente evidencia experimental y estadística.

Las estrategias más utilizadas son **Hard Filtering** y **Variant Quality Score Recalibration (VQSR)**.

---

## 4.17.1 Hard Filtering

El Hard Filtering aplica umbrales fijos sobre diferentes métricas de calidad calculadas durante el llamado de variantes.

Es el método recomendado cuando:

- se dispone de pocas muestras;
- no existen bases de datos de variantes conocidas;
- se trabaja con organismos no modelo;
- no es posible entrenar modelos estadísticos complejos.

Las principales métricas utilizadas incluyen:

### Quality by Depth (QD)

Representa la calidad de la variante normalizada por la profundidad de cobertura.

Valores bajos indican que la aparente calidad de la variante depende únicamente de una cobertura muy elevada.

---

### Fisher Strand Bias (FS)

Evalúa si existe un sesgo significativo entre las lecturas provenientes de ambas hebras del ADN.

Un valor elevado puede indicar errores sistemáticos de secuenciación o alineamiento.

---

### Strand Odds Ratio (SOR)

Constituye una medida alternativa del sesgo de hebra menos sensible a diferencias extremas en la cobertura.

---

### Mapping Quality (MQ)

Corresponde a la calidad promedio del alineamiento de todas las lecturas que soportan una variante.

Valores bajos suelen indicar regiones repetitivas o alineamientos ambiguos.

---

### Mapping Quality Rank Sum Test (MQRankSum)

Compara la calidad del alineamiento entre las lecturas que contienen el alelo de referencia y aquellas que contienen el alelo alternativo.

Grandes diferencias pueden sugerir errores de mapeo.

---

### Read Position Rank Sum Test

Evalúa si el alelo alternativo aparece preferentemente en los extremos de las lecturas.

Las variantes verdaderas suelen distribuirse uniformemente a lo largo de toda la longitud de la lectura.

---

### Depth (DP)

Representa el número total de lecturas que cubren una posición determinada.

Coberturas extremadamente bajas disminuyen la confianza en el genotipo, mientras que coberturas excesivamente altas pueden indicar duplicados o regiones repetitivas.

---

## 4.17.2 Variant Quality Score Recalibration (VQSR)

Cuando se dispone de grandes cohortes y bases de datos de variantes conocidas, GATK recomienda utilizar **Variant Quality Score Recalibration (VQSR)**.

Este procedimiento utiliza modelos de aprendizaje automático basados en mezclas de gaussianas para aprender las características de variantes verdaderas a partir de conjuntos de entrenamiento como dbSNP, HapMap o 1000 Genomes.

Posteriormente, cada variante recibe una puntuación que representa la probabilidad de corresponder a una variante real.

Las principales ventajas de VQSR son:

- mayor sensibilidad;
- menor tasa de falsos positivos;
- filtrado adaptativo;
- mejor desempeño en estudios poblacionales.

No obstante, este método requiere un gran número de variantes conocidas y un tamaño de muestra considerable, por lo que generalmente no resulta adecuado para organismos no modelo.

---

# 4.18 Formato Variant Call Format (VCF)

Una vez finalizado el llamado y filtrado de variantes, los resultados se almacenan en archivos **VCF (Variant Call Format)**, considerados el estándar internacional para representar variantes genómicas.

El formato VCF fue desarrollado para facilitar el intercambio de información entre diferentes herramientas bioinformáticas y contiene tanto información de cada variante como los genotipos de todos los individuos analizados.

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

# 4.20 Consideraciones generales

La precisión del llamado de variantes depende directamente de la calidad de todas las etapas previas del análisis. Un alineamiento incorrecto, una cobertura insuficiente o la presencia de errores sistemáticos pueden generar un elevado número de falsos positivos o falsos negativos.

Por ello, el análisis de datos NGS debe entenderse como un flujo de trabajo integrado, donde cada etapa influye sobre la siguiente. La utilización de herramientas robustas como Bowtie2 para el alineamiento y GATK para el llamado de variantes, junto con procedimientos adecuados de control de calidad y filtrado, permite obtener conjuntos de variantes altamente confiables para estudios de genética molecular, genética de poblaciones, evolución y mejoramiento genético.

---

# Referencias bibliográficas (APA 7.ª edición)

- Broad Institute. (2024). *Genome Analysis Toolkit (GATK) Best Practices*. https://gatk.broadinstitute.org/

- Danecek, P., Bonfield, J. K., Liddle, J., Marshall, J., Ohan, V., Pollard, M. O., ... Li, H. (2021). Twelve years of SAMtools and BCFtools. *GigaScience, 10*(2), giab008.

- Langmead, B., Trapnell, C., Pop, M., & Salzberg, S. L. (2009). Ultrafast and memory-efficient alignment of short DNA sequences to the human genome. *Genome Biology, 10*(3), R25.

- Langmead, B., & Salzberg, S. L. (2012). Fast gapped-read alignment with Bowtie 2. *Nature Methods, 9*(4), 357–359.

- Li, H., & Durbin, R. (2009). Fast and accurate short read alignment with Burrows–Wheeler Transform. *Bioinformatics, 25*(14), 1754–1760.

- Li, H. (2011). A statistical framework for SNP calling, mutation discovery and population genetic parameter estimation from sequencing data. *Bioinformatics, 27*(21), 2987–2993.

- McKenna, A., Hanna, M., Banks, E., Sivachenko, A., Cibulskis, K., Kernytsky, A., ... DePristo, M. A. (2010). The Genome Analysis Toolkit: A MapReduce framework for analyzing next-generation DNA sequencing data. *Genome Research, 20*(9), 1297–1303.

- Van der Auwera, G. A., & O'Connor, B. D. (2020). *Genomics in the Cloud*. O'Reilly Media.
