# Restriction-site Associated DNA Sequencing (RADseq)

> Una guía sobre los métodos de secuenciación de representación reducida del genoma basados en enzimas de restricción y un flujo de trabajo completo de ddRADseq

---

# Introducción

La **Restriction-site Associated DNA Sequencing (RADseq)** es una familia de técnicas de secuenciación de representación reducida (*Reduced Representation Sequencing, RRS*) que permitieron obtener miles de SNPs distribuidos por todo el genoma de forma reproducible y a bajo costo (Baird et al., 2008). Su principio se basa en el corte del ADN genómico con enzimas de restricción, seguido de la secuenciación de los fragmentos adyacentes a los sitios de corte, generando un muestreo sistemático del genoma.

Las metodologías de RADseq resultaron ser especialmente útiles en organismos no modelo, donde no existe un genoma de referencia o el costo de un WGS resulta prohibitivo. Los marcadores RAD se implementaron inicialmente mediante microarrays y posteriormente se adaptaron para NGS (secuenciación de próxima generación). Por lo que existen varias adaptaciones diseñadas para resolver desafíos específicos de fragmentación y cobertura de datos.

La variante **ddRADseq** (*double digest* RADseq; Peterson et al., 2012) reemplaza la fragmentación mecánica del RADseq original (Baird et al., 2008) por una segunda enzima de restricción. Solo se secuencian los fragmentos delimitados por ambas enzimas que cumplen un rango de tamaño determinado, lo que aumenta la reproducibilidad entre muestras y reduce el costo. Esta variante es actualmente una de las más utilizadas en estudios de genómica de poblaciones.

En este módulo se simularon datos ddRADseq a partir de un fragmento del genoma de referencia de *Solanum lycopersicum* Micro-Tom (cromosoma 6, primeros 10 Mb; AP028940.1; GCF_036512215.1 SLM_r2.1), usando RADinitio. Las lecturas simuladas fueron procesadas con ipyrad para el ensamblaje de loci y la identificación de variantes. Finalmente, se estimaron estadísticas de diversidad genética —diversidad nucleotídica (π), heterocigosidad observada y diferenciación genética (F_ST) con VCFtools.

---

## ¿Cómo funciona RADseq?

Aunque existen diferentes variantes, todas comparten el mismo principio:

1. Extracción del ADN genómico.
2. Digestión del ADN con una o varias enzimas de restricción.
3. Ligación de adaptadores a los extremos generados por las enzimas.
4. Selección de una fracción del genoma (mediante tamaño o combinaciones de enzimas).
5. Amplificación mediante PCR.
6. Secuenciación en plataformas Illumina.
7. Identificación de SNPs y otros polimorfismos.
   
---
<p align="center">
  <img width="720" height="1280" alt="image" src="https://github.com/user-attachments/assets/b3c91d55-c2f1-414f-a477-d907fc421683" />

</p>

<p align="center">
<b>Figura 1.</b> Esquema general de la preparación de bibliotecas basadas en sitios de restricción (RADseq). El diagrama resume las etapas comunes a la familia de métodos RADseq. Dependiendo del protocolo empleado, la reducción adicional de la complejidad del genoma puede realizarse mediante fragmentación mecánica (RADseq original) o mediante una segunda digestión con enzimas de restricción (ddRADseq).
</p>

---

## ¿Por qué utilizar RADseq?

Las técnicas RADseq surgieron para resolver un problema común en genómica: secuenciar genomas completos de muchos individuos puede ser innecesario y muy costoso. En muchos estudios de genética de poblaciones, evolución o filogenómica, basta con analizar miles de marcadores distribuidos por todo el genoma para responder preguntas biológicas. RADseq permite obtener esos marcadores de manera eficiente y reproducible, multiplexar numerosos individuos en una misma corrida de secuenciación y reducir considerablemente el costo respecto al WGS.

---

## Los métodos RADseq

Desde la publicación del protocolo original (Baird et al., 2008), se han desarrollado numerosas variantes que buscan simplificar el protocolo, disminuir mejorar la reproducibilidad, ampliar la cobertura, o resolver algún desafio en especifico. Cada variante modifica alguna etapa del procedimiento, como el número de enzimas utilizadas, la forma de seleccionar los fragmentos o el método de preparación de bibliotecas.

La elección del método depende del organismo de estudio, el presupuesto disponible, el tamaño del genoma y la pregunta biológica.

| Método | Año | Descripción |
|---------|:---:|-------------|
| **RRL (Reduced Representation Libraries)** | Van Tassell et al. (2008) | Es el primer método formal de representación reducida. Se utiliza una sola enzima de restricción de corte poco frecuente (p. ej., BglII , EcoRI ) para digerir el ADN genómico. Los fragmentos resultantes se separan por tamaño mediante electroforesis y se selecciona una fracción específica antes de construir la biblioteca de secuenciación. Solo esa fracción de tamaño seleccionada es secuenciada. Cayo en desuso puesto que la selección de tamaño por gel es un paso manual, laborioso y con baja reproducibilidad entre experimentos. Solo usa una enzima, lo que limita el control sobre el número de loci obtenidos. |
| **RADseq** | Baird et al. (2008) | Es el protocolo fundamental de la familia RADseq. El ADN genómico se digiere con una enzima de restricción de corte raro (p. ej., SbfI ). A los fragmentos resultantes se les ligan adaptadores específicos con secuencias índice (códigos de barras) para multiplexar muestras. Luego el ADN es fragmentado mecánicamente por sonicación a un tamaño uniforme, y solo los fragmentos que contienen el sitio de restricción (con el adaptador ligado) son capturados y secuenciados. Esto genera lecturas "tipo P1" que comienzan exactamente en el sitio de restricción. La principal limitación radica en que la sonicación introduce variabilidad y los fragmentos P2 (el extremo opuesto) tienen profundidad de cobertura variable. Requiere más pasos de laboratorio que variantes posteriores. |
| **GBS (Genotyping-by-Sequencing)** | Elshire et al. (2011) | Simplifica radicalmente el protocolo RADseq. Usa una sola enzima de restricción sensible a la metilación (típicamente ApeKI o PstI ) que corta preferentemente en regiones de baja metilación, es decir, regiones génicas activas. Elimina completamente la sonicación y la selección física de tamaño: la PCR favorece naturalmente los fragmentos más cortos, reduciendo la complejidad del genoma sin pasos adicionales. Los adaptadores están ligados directamente a los extremos de restricción. Es el método preferido cuando se necesita genotipar cientos o millas de individuos a bajo costo. Los loci obtenidos varían más entre muestras que en ddRADseq y no es ideal para organismos con genomas muy grandes o con alto contenido de secuencias repetitivas. |
| **CRoPS (Complexity Reduction of Polymorphic Sequences)** | Van Orsouw et al. (2007) | Uno de los primeros enfoques de doble digestión. Utiliza dos enzimas de restricción —una de corte rara y una de corte frecuente— para reducir la complejidad del genoma de forma más controlada que GBS. Fue desarrollado originalmente para la plataforma de secuenciación 454 (pirosecuenciación), aunque luego se adaptó a Illumina. Solo se secuencian los fragmentos delimitados por ambas enzimas. Ha sido en gran medida reemplazado por ddRADseq. |
| **2bRAD** | Wang et al. (2012) | Utiliza enzimas de restricción tipo IIB (p. ej., BsaXI , BcgI ), que tienen la propiedad única de cortar a una distancia fija a ambos lados de su sitio de reconocimiento. Esto genera fragmentos de longitud absolutamente uniforme (~33–36 pb), independientemente del genoma estudiado. Los fragmentos son ligados a adaptadores y secuenciados directamente. Ha sido frecuentemente usada en genómica de poblaciones en organismos marinos y acuáticos (corales, ostras, peces), donde la uniformidad de los fragmentos simplifica el análisis bioinformático. Pero tambien, las lecturas muy cortas (33–36 pb) limitan la capacidad de mapeo único en genomas con alta proporción de secuencias repetitivas. La cobertura por locus puede ser más baja que en otros métodos.  |
| **ddRADseq (Double Digest RADseq)** | Peterson et al. (2012) | Es actualmente la variante más utilizada en genómica de poblaciones. Sustituye la sonicación del RADseq original por una segunda enzima de restricción de corte frecuente (p. ej., EcoRI + MseI, o SbfI + MspI). Solo se secuencian los fragmentos delimitados en un extremo por la enzima rara y en el otro por la enzima frecuente, dentro de un rango de tamaño mediante electroforesis seleccionada en gel o cromatografía. Esto permite controlar con precisión el número de loci obtenidos y mejorar enormemente la reproducibilidad entre experimentos y laboratorios. Requiere optimización de la combinación de enzimas según el tamaño y el contenido de GC del genoma. |
| **MSG (Multiplexed Shotgun Genotyping)** | Andolfatto et al. (2011) | Diseñado originalmente para la construcción de mapas genéticos y estudios de genética de poblaciones con multiplexación masiva. Utiliza una enzima de restricción y simplifica varios pasos del protocolo original, permitiendo procesar un número muy elevado de individuos simultáneamente mediante indexación dual. Su nombre refleja el enfoque de "shotgun" (escopeta): secuencia una fracción aleatoria del genoma en lugar de loci específicos. Es utilizado para mapeo genético de alta densidad, genotipado masivo en Drosophila y otros organismos modelo, estudios de selección artificial y de asociación genoma-fenotipo (GWAS). |
| **SBG (Sequence-Based Genotyping)** | Truong et al. (2012) | Combina una enzima de corte raro y una de corte frecuente para seleccionar un subconjunto específico y reducido de loci distribuidos por el genoma. El diseño busca un equilibrio entre reducción de complejidad y representación uniforme del genoma, con mayor control sobre el número final de loci que GBS pero menor que ddRADseq. Menos flexible que ddRADseq en la selección del número de loci. Su uso está más concentrado en el ámbito del mejoramiento que en la genómica evolutiva. |
| **ezRAD** | Toonen et al. (2013) | Fue diseñado específicamente para facilitar la implementación de RADseq en organismos no modelo sin acceso a equipamiento especializado. Utiliza una o dos enzimas de restricción de corte frecuente (p. ej., DpnII , MboI ), pero en lugar de adaptadores RAD especializados, emplea kits comerciales estándar de preparación de bibliotecas Illumina (p. ej., TruSeq). Los fragmentos de restricción de menor tamaño son secuenciados directamente sin selección física adicional. La ausencia de selección de tamaño y el uso de enzimas de corte frecuentes puede generar un número muy elevado de lugares con cobertura desigual. Menor reproducibilidad entre experimentos que ddRADseq. |

---

# GBS y ddRADseq: dos estrategias para un mismo objetivo

## GBS — Genotyping-by-Sequencing

GBS fue desarrollado como una alternativa radicalmente simplificada a RADseq, diseñada para hacer accesible el genotipado masivo de SNPs en plantas de cultivo con genomas grandes y alta diversidad. Su principio central es la reducción de la complejidad del genoma mediante digestión con enzimas de restricción sensibles a la metilación del ADN, aprovechando el hecho de que las regiones génicas activas (eucromatina) están menos metiladas que las regiones heterocromáticas y repetitivas.

El protocolo original consta de los siguientes pasos: 

1. Digestión enzimática: El ADN genómico de cada muestra es digerido con una enzima de restricción sensible a la metilación. Las enzimas más utilizadas son:

| Enzima | Sitio de reconocimiento | Características |
|---------|-------------------------|-----------------|
| **ApeKI** | G/CWCG | Enzima del protocolo original. Reconoce sitios poco metilados y dirige el muestreo hacia regiones génicas. |
| **PstI** | CTGCA/G | Muy utilizada en gramíneas y otras plantas. |
| **EcoRI** | G/AATTC | Utilizada según el tamaño y complejidad del genoma. |
| **HindIII** | A/AGCTT | Alternativa para distintos diseños experimentales. |
| **MseI** | T/TAA | Se emplea principalmente en protocolos de doble digestión. |

2. Ligación de adaptadores: A los extremos de restricción generados se ligan dos tipos de adaptadores:

Adaptador común (Y-adapter): ligado al extremo de corte frecuente, impide la amplificación de fragmentos con dos adaptadores comunes.
Adaptador con barcode: contiene una secuencia índice única por muestra (4–8 pb), lo que permite multiplexar múltiples muestras en una sola carrera de secuenciación.

3. Combinación de muestras y limpieza: Las muestras se combinan (pool) y se purifica el ADN para eliminar fragmentos muy pequeños y reactivos.
   
4. Amplificación por PCR: Se amplifica la biblioteca con primers universales. A diferencia de ddRADseq, no se realiza selección de tamaño por gel o cromatografía. La PCR favorece naturalmente los fragmentos más cortos, reduciendo la complejidad del genoma sin pasos adicionales. Los fragmentos más largos amplifican con menor eficiencia y están subrepresentados en la biblioteca final. Los fragmentos cortos (<600 pb) amplifican más eficientemente, actuando como un filtro implícito de tamaño.

5. Secuenciación: La biblioteca se secuencia en plataformas Illumina (típicamente en modo single-end, aunque el paired-end también se usa). Las lecturas comienzan exactamente en el sitio de restricción, lo que facilita su identificación y el proceso de demultiplexación. 

El éxito de GBS impulsó el desarrollo de variantes que buscan resolver alguna de sus limitaciones. **2-enzyme GBS (2e-GBS)**: Usa dos enzimas para mejorar la reproducibilidad entre experimentos, acercándose al concepto de ddRADseq pero manteniendo la simplicidad del protocolo **GBS. nextRAD**: Utiliza primers de PCR con extensión en el extremo 3' que se alinean específicamente con el sitio de restricción, aumentando la especificidad del muestreo. **rapture (RAD capture)**: Combina GBS con captura por hibridación para enriquecer loci específicos de interés, integrando las ventajas de GBS y de los paneles de captura dirigida.

## ddRADseq — Double Digest RADseq

ddRADseq fue desarrollado como una alternativa más económica y flexible al RADseq original, eliminando la necesidad de fragmentación mecánica (sonicación) y sustituyéndola por una doble digestión enzimática combinada con selección de tamaño estricta. Su principio central es la reducción de la complejidad del genoma mediante el uso simultáneo de dos enzimas de restricción con frecuencias de corte distintas, lo que permite un control más preciso sobre la densidad de marcadores y el tamaño de la biblioteca final.

El protocolo original consta de los siguientes pasos:

1. Digestión enzimática doble: El ADN genómico de cada muestra es digerido simultáneamente con dos enzimas de restricción: una de corte raro (rare cutter) y una de corte frecuente (common cutter). Las combinaciones más utilizadas son:

| Enzima | Sitio de reconocimiento | Características |
|---------|--------------------------|-----------------|
| **SbfI** | CCTGCA/GG | Enzima de corte raro (8 pb), genera pocos fragmentos, común en la combinación original de Peterson et al. (2012). |
| **EcoRI** | G/AATTC | Enzima de corte raro alternativa, usada en combinación con enzimas de corte frecuente. |
| **MspI** | C/CGG | Enzima de corte frecuente, sensible al contexto de metilación CpG. |
| **MseI** | T/TAA | Enzima de corte frecuente (4 pb), genera un gran número de sitios de corte. |
| **NlaIII** | CATG/C | Enzima de corte frecuente, utilizada en combinaciones alternativas según el genoma objetivo. |

La combinación rara + frecuente asegura que los fragmentos generados provengan de regiones flanqueadas por un sitio raro en un extremo y un sitio frecuente en el otro, lo cual reduce drásticamente el número de loci muestreados en comparación con usar una sola enzima.

2. Ligación de adaptadores: A cada tipo de extremo de corte se liga un adaptador específico:

Adaptador P1: se liga al extremo generado por la enzima de corte raro; contiene el barcode único por muestra y el sitio de unión para la secuenciación.
Adaptador P2: se liga al extremo generado por la enzima de corte frecuente; frecuentemente incluye un índice adicional, permitindo doble indexación (barcode + índice) para multiplexar muchas más muestras que en GBS.

3. Combinación de muestras y selección de tamaño por gel: Las muestras se combinan (pool) y, a diferencia de GBS, se realiza una selección de tamaño física y precisa de los fragmentos, típicamente mediante electroforesis en gel de agarosa o sistemas automatizados (por ejemplo, Pippin Prep/BluePippin). Se selecciona una ventana estrecha de tamaño (por ejemplo 300–400 pb), lo cual es el paso clave que diferencia a ddRADseq de GBS: en lugar de dejar que la PCR filtre implícitamente por tamaño, aquí el tamaño se controla directamente, mejorando la reproducibilidad entre librerías y experimentos.

4. Amplificación por PCR: Se amplifica la biblioteca ya seleccionada por tamaño con primers universales. Como el rango de tamaños ya es angosto y controlado, la PCR introduce mucho menos sesgo diferencial entre fragmentos largos y cortos, comparado con GBS.
   
5. Secuenciación: La biblioteca se secuencia en plataformas Illumina, generalmente en modo paired-end, lo cual permite recuperar información de ambos extremos del fragmento (útil para ensamblar loci más largos y mejorar la llamada de variantes). Las lecturas comienzan en los sitios de restricción de ambas enzimas, facilitando la identificación de loci homólogos entre muestras.
   
El uso extendido de ddRADseq impulsó variantes y protocolos derivados que buscan resolver algunas de sus limitaciones. **3RAD:** Añade una tercera enzima de restricción y usa adaptadores con sitios de reconocimiento internos, lo que reduce la formación de dímeros de adaptador y mejora la eficiencia de la ligación, un problema común en ddRADseq estándar. **ezRAD:** Simplifica el protocolo utilizando enzimas de restricción comunes en kits de preparación de librerías estándar (sin necesidad de adaptadores personalizados), sacrificando algo de control sobre la reducción de complejidad a cambio de simplicidad de laboratorio. **BestRAD:** Incorpora biotina en los adaptadores, permitiendo la captura de fragmentos con beads magnéticas; es especialmente útil para muestras de baja calidad o de fuentes no invasivas (ej. ADN ambiental o de heces).

---

# Otros Métodos de reducción de la complejidad genómica: Multiplex PCR y Target Enrichment

## Multiplex PCR

Multiplex PCR es una estrategia de reducción de la complejidad del genoma que, a diferencia de las tecnologias ddRADseq, no depende de enzimas de restricción sino de la amplificación dirigida y simultánea de múltiples regiones genómicas específicas mediante el diseño de varios pares de primers en una sola reacción. Su principio central es la selección a priori de los loci de interés (genes candidatos, regiones microsatélite, SNPs previamente identificados), lo que la convierte en un método altamente dirigido en contraste con los métodos de reducción "al azar" como ddRADseq.

El protocolo original consta de los siguientes pasos: 

1. Diseño de primers: Se diseñan múltiples pares de primers específicos para las regiones genómicas de interés, cuidando evitar la formación de dímeros de primers y asegurando temperaturas de alineamiento (Tm) compatibles entre todos los pares incluidos en la reacción.
2. Amplificación simultánea: El ADN genómico de cada muestra se somete a una única reacción de PCR que contiene todos los pares de primers, amplificando simultáneamente decenas a cientos de loci en un solo tubo. Es requerido realizar un balanceo de la redacción, ajustando las concentraciones relativas de cada par de primers para evitar que los amplicones más eficientes agoten los reactivos, un fenómeno conocido como competencia de amplificación (amplicon dropout de los loci menos eficientes).
3. Adición de barcodes: Mediante una segunda ronda de PCR (con primers de fusión) o mediante ligación posterior, se añaden los índices/barcodes de muestra y los adaptadores de secuenciación. Secuenciación: La biblioteca resultante se secuencia en plataformas Illumina, típicamente en modo paired-end, dado que los amplicones suelen tener tamaños conocidos y relativamente uniformes, lo que facilita el ensamblaje y la llamada de variantes en las regiones dirigidas.
4. Multiplex PCR es ampliamente usado en paneles de genotipado de baja a media densidad (por ejemplo, paneles de identificación de variedades o control de pedigrí), en estudios donde ya existe un conjunto conocido de SNPs o genes candidatos de interés, y en contextos donde el bajo costo por muestra y la rapidez son más importantes que la cobertura genómica amplia. Su principal limitación es el número de loci que pueden multiplexarse simultáneamente sin generar interferencia entre primers (usualmente cientos, no miles), lo que la hace menos adecuada para estudios de genómica poblacional a gran escala en comparación con ddRADseq.

## Target Enrichment

Target Enrichment (captura de secuencias dirigidas) es una estrategia de reducción de la complejidad genómica basada en hibridación selectiva de fragmentos de ADN mediante sondas (probes/baits) diseñadas específicamente para regiones de interés, permitiendo enriquecer miles de loci simultáneamente con mayor especificidad que Multiplex PCR y mayor flexibilidad de diseño que los métodos basados en enzimas de restricción. Su principio central es la hibridación en solución (o en array) entre sondas biotiniladas complementarias a las regiones objetivo y los fragmentos genómicos de la biblioteca, seguida de la captura física de los híbridos mediante beads magnéticas de estreptavidina.

El protocolo original consta de los siguientes pasos: 

1. Preparación de la biblioteca genómica: El ADN genómico de cada muestra se fragmenta (mecánicamente por sonicación o enzimáticamente) y se prepara una biblioteca estándar de secuenciación, con ligación de adaptadores y barcodes por muestra, de manera similar a una biblioteca de WGS convencional.
2. Diseño de sondas (baits): Se diseñan sondas de ADN o ARN biotiniladas, de aproximadamente 80–120 pb, complementarias a las regiones genómicas de interés (genes candidatos, exones, regiones conservadas o loci ortólogos entre especies).
3. Hibridación en solución: Las bibliotecas genómicas de las muestras (a menudo ya multiplexadas) se hibridan en solución con el pool de sondas biotiniladas durante 16–24 horas, permitiendo que las sondas se unan específicamente a los fragmentos complementarios.
4. Captura con beads magnéticas: Los híbridos sonda-fragmento se capturan utilizando beads magnéticas recubiertas de estreptavidina, que se unen a la biotina de las sondas; los fragmentos no unidos (regiones no objetivo) se lavan y se descartan.
5. Amplificación por PCR: Los fragmentos capturados y enriquecidos se amplifican mediante PCR para generar suficiente material para la secuenciación.
6. Secuenciación: La biblioteca enriquecida se secuencia en plataformas Illumina, típicamente en modo paired-end, alcanzando una cobertura de profundidad mucho mayor en las regiones objetivo comparado con una secuenciación de genoma completo sin enriquecer.
   
El uso extendido de Target Enrichment impulsó variantes orientadas a distintos objetivos de investigación. **Exome capture:** Enfocado exclusivamente en las regiones codificantes (exoma), ampliamente usado en estudios humanos y de especies modelo con anotaciones genómicas robustas. **Anchored hybrid enrichment (AHE):** Diseñado para estudios filogenéticos profundos, dirigido a regiones conservadas flanqueadas por regiones variables, útil para resolver relaciones entre especies muy divergentes. **Sequence capture for phylogenomics (por ejemplo, kits Angiosperms353):** Diseñado específicamente para capturar cientos de genes de copia única en plantas, permitiendo estudios filogenómicos comparables entre especies distantes.

Target Enrichment: Es el método que más ha crecido en la última década, particularmente en filogenómica (por ejemplo, kits como Angiosperms353, o Hyb-Seq en general) y en estudios de genes candidatos bajo selección, porque permite obtener cobertura muy profunda y consistente en cientos o miles de genes específicos, comparable entre especies distantes, algo que RADseq no logra bien a escalas filogenéticas profundas por la degradación de sitios homólogos entre especies muy divergentes. También domina en aplicaciones clínicas humanas (paneles de genes de enfermedad, exoma completo) y en estudios evolutivos donde se necesita ortología clara entre especies. Su limitación sigue siendo el costo de diseño inicial de las sondas y que solo se captura lo que se decide capturar, sin posibilidad de descubrir variación fuera del panel diseñado.

<p align="center">
  <img width="597" height="878" alt="image" src="https://github.com/user-attachments/assets/226dfccf-74e0-4e9e-8a23-a224a137d7c8" />

</p>

<p align="center">
<b>Figura 2.</b> Esquema del proceso de preparación de bibliotecas mediante Target Enrichment (Sequence Capture) para secuenciación Illumina. El ADN genómico es fragmentado, las regiones objetivo son capturadas mediante sondas biotiniladas y esferas de estreptavidina, posteriormente se realiza la reparación de extremos, la ligación de adaptadores (incluyendo un Unique Molecular Identifier, UMI) y la amplificación por PCR, durante la cual se incorporan los índices de muestra (sample indexes) para la multiplexación.
</p>

---

# Un flujo de trabajo completo de ddRADseq

## Entorno computacional

Todos los análisis se realizaron en Ubuntu bajo WSL2 (Windows Subsystem for Linux) localmente, usando el gestor de paquetes conda (Miniforge3).

```bash
# Creación del ambiente conda
conda create -n radseq2 python=3.10 -y
conda activate radseq2

# Instalación de herramientas
conda install -c bioconda -c conda-forge ipyrad samtools vcftools bbtools -y
pip install setuptools==69.0.0
pip install radinitio
```
---

## 1. Obtención y preparación del genoma de referencia

Se descargó la secuencia del cromosoma 6 del genoma de referencia de *Solanum lycopersicum* Micro-Tom (AP028940.1; 52,172,941 pb) desde NCBI en formato FASTA. Para reducir el tiempo de cómputo, se extrajo un fragmento de los primeros 10 Mb con `samtools faidx`:

```bash
# Indexar el genoma
samtools faidx chr6_SLM.fasta

# Extraer los primeros 10 Mb
samtools faidx chr6_SLM.fasta "AP028940.1:1-10000000" > chr6_10Mb.fasta

# Verificar el fragmento
ls -lh chr6_10Mb.fasta
# -rw-r--r-- 1 user 9.7M chr6_10Mb.fasta
```
---

## 2. Simulación de datos ddRADseq con RADinitio

RADinitio v1.2.3 simula el proceso completo de preparación de una biblioteca ddRADseq a partir de un genoma de referencia. Internamente utiliza `msprime` para simular la historia demográfica y la variación genética bajo un modelo coalescente, y luego simula la digestión enzimática, la selección de fragmentos por tamaño, la amplificación por PCR y la secuenciación.

### 2.1 Preparación de archivos de entrada

```bash
# Crear lista de cromosomas/scaffolds del fragmento
awk 'sub(/^>/, "")' chr6_10Mb.fasta | head -1 > chrom_list.txt
cat chrom_list.txt
# AP028940.1:1-10000000

# Crear directorio de salida
mkdir sim_reads
```

### 2.2 Ejecución de la simulación

```bash
radinitio --simulate-all \
    --genome chr6_10Mb.fasta \          # Genoma de referencia
    --chromosomes chrom_list.txt \       # Lista de cromosomas a simular
    --out-dir sim_reads \                # Directorio de salida
    --n-pops 3 \                         # Número de poblaciones
    --pop-eff-size 1000 \                # Tamaño efectivo poblacional (Ne)
    --n-seq-indv 5 \                     # Individuos por población
    --library-type ddRAD \               # Tipo de biblioteca
    --enz ecoRI \                        # Enzima de restricción 1
    --enz2 mseI \                        # Enzima de restricción 2
    --insert-mean 350 \                  # Tamaño medio de inserto (pb)
    --insert-stdev 35 \                  # Desviación estándar del inserto
    --pcr-cycles 9 \                     # Ciclos de PCR
    --coverage 10 \                      # Cobertura objetivo (X)
    --read-length 150                    # Longitud de lectura (pb)
```

**Parámetros del modelo demográfico (msprime):**

| Parámetro | Valor |
|-----------|-------|
| Tasa de mutación | 7 × 10⁻⁸ |
| Número de poblaciones | 3 |
| Ne por población | 1,000 |
| Individuos por población | 5 |
| Tasa de migración | 5 × 10⁻⁴ (simétrica entre todas las poblaciones) |
| Modelo de sustitución | equal (Jukes-Cantor) |
| Probabilidad de INDEL | 0.01 |

**Parámetros de la biblioteca ddRAD:**

| Parámetro | Valor |
|-----------|-------|
| Enzima 1 | EcoRI (G/AATTC) |
| Enzima 2 | MseI (T/TAA) |
| Tamaño de inserto (media ± SD) | 350 ± 35 pb |
| Rango de tamaño seleccionado | 280–420 pb |
| Longitud de lectura | 150 pb |
| Cobertura | 10X |
| Longitud base del locus | 1,000 pb |
| Distancia mínima entre loci | 1,000 pb |

### 2.3 Conversión de FASTA a FASTQ con BBtools

ipyrad requiere lecturas en formato FASTQ. Los archivos simulados por RADinitio están en formato FASTA comprimido y se convierten con `reformat.sh` de BBtools:

```bash
mkdir ddrad_fq
cd sim_reads/rad_reads/

# Loop de conversión para todas las muestras (R1 y R2)
for i in msp_*.1.fa.gz; do
    reformat.sh \
        in=${i//.1./.#.} \
        out=../../ddrad_fq/${i//.1.fa.gz/_R#_.fastq.gz}
done
```

Este loop genera pares de archivos FASTQ (R1 y R2) para cada uno de los 15 individuos simulados, resultando en **30 archivos FASTQ**.

---

## 3. Procesamiento de lecturas con ipyrad

ipyrad v0.9.108 es una plataforma para el ensamblaje de loci RAD y la identificación de variantes. El flujo de trabajo consta de 7 pasos: (1) carga de datos, (2) filtrado y trimming, (3) clustering/mapeo intramuestral, (4) estimación de tasa de error y heterocigosidad, (5) llamado de bases, (6) clustering intermuestral, y (7) filtrado y formateo de salida.

### 3.1 Creación del archivo de parámetros

```bash
cd ~/radseq_practica
ipyrad -n sim_data
# Genera: params-sim_data.txt
```

### 3.2 Configuración de parámetros

El archivo `params-sim_data.txt` fue editado con los siguientes valores:

```
sim_data                                              ## [0]  assembly_name
/home/user/radseq_practica/proc_reads                 ## [1]  project_dir
                                                      ## [2]  raw_fastq_path
                                                      ## [3]  barcodes_path
/home/user/radseq_practica/ddrad_fq/*.fastq.gz        ## [4]  sorted_fastq_path
denovo                                                ## [5]  assembly_method
                                                      ## [6]  reference_sequence
pairddrad                                             ## [7]  datatype
AATTC,TAA                                             ## [8]  restriction_overhang
5                                                     ## [9]  max_low_qual_bases
33                                                    ## [10] phred_Qscore_offset
6                                                     ## [11] mindepth_statistical
6                                                     ## [12] mindepth_majrule
10000                                                 ## [13] maxdepth
0.85                                                  ## [14] clust_threshold
0                                                     ## [15] max_barcode_mismatch
2                                                     ## [16] filter_adapters
35                                                    ## [17] filter_min_trim_len
2                                                     ## [18] max_alleles_consens
0.05                                                  ## [19] max_Ns_consens
0.05                                                  ## [20] max_Hs_consens
4                                                     ## [21] min_samples_locus
0.2                                                   ## [22] max_SNPs_locus
8                                                     ## [23] max_Indels_locus
0.5                                                   ## [24] max_shared_Hs_locus
0, 0, 0, 0                                            ## [25] trim_reads
0, 0, 0, 0                                            ## [26] trim_loci
v                                                     ## [27] output_formats (VCF)
```

### 3.3 Ejecución del pipeline completo

```bash
# Correr los 7 pasos con 4 núcleos
ipyrad -p params-sim_data.txt -s 1234567 -c 4
```

---

## 4. Filtrado de SNPs con VCFtools

El VCF generado por ipyrad fue filtrado para eliminar INDELs y aplicar umbrales de calidad y profundidad de cobertura:

```bash
vcftools --vcf ~/radseq_practica/proc_reads/sim_data_outfiles/sim_data.vcf \
    --remove-indels \        # Eliminar inserciones y deleciones
    --max-missing 0.8 \      # Máximo 20% de datos faltantes por sitio
    --min-meanDP 6 \         # Profundidad media mínima de 6X
    --max-meanDP 100 \       # Profundidad media máxima de 100X
    --recode \               # Generar VCF filtrado
    --out ~/radseq_practica/sim_data_filtered
```

---

## 5. Estimación de estadísticas de diversidad genética

Se definieron tres grupos poblacionales a partir del archivo `popmap.tsv` generado por RADinitio:

```bash
# Crear archivos de población para VCFtools
grep "pop0" sim_reads/popmap.tsv | cut -f1 > pop0.txt  # msp_00 a msp_04
grep "pop1" sim_reads/popmap.tsv | cut -f1 > pop1.txt  # msp_05 a msp_09
grep "pop2" sim_reads/popmap.tsv | cut -f1 > pop2.txt  # msp_10 a msp_14
```

### 5.1 Diversidad nucleotídica (π) por sitio

```bash
# π por sitio para cada población
for pop in pop0 pop1 pop2; do
    vcftools --vcf sim_data_filtered.recode.vcf \
        --keep ${pop}.txt \
        --site-pi \
        --out pi_${pop}
done

# Calcular π promedio por población
awk 'NR>1 {sum+=$3; n++} END {print "pi_pop0 =", sum/n}' pi_pop0.sites.pi
awk 'NR>1 {sum+=$3; n++} END {print "pi_pop1 =", sum/n}' pi_pop1.sites.pi
awk 'NR>1 {sum+=$3; n++} END {print "pi_pop2 =", sum/n}' pi_pop2.sites.pi
```

### 5.2 Heterocigosidad observada

```bash
vcftools --vcf sim_data_filtered.recode.vcf \
    --het \
    --out heterocigosidad
# Salida: heterocigosidad.het (columnas: INDV, O(HOM), E(HOM), N_SITES, F)
```

### 5.3 Diferenciación genética (F_ST de Weir y Cockerham)

```bash
# Comparación pop0 vs pop1
vcftools --vcf sim_data_filtered.recode.vcf \
    --weir-fst-pop pop0.txt \
    --weir-fst-pop pop1.txt \
    --out fst_pop0_pop1

# Comparación pop0 vs pop2
vcftools --vcf sim_data_filtered.recode.vcf \
    --weir-fst-pop pop0.txt \
    --weir-fst-pop pop2.txt \
    --out fst_pop0_pop2

# Comparación pop1 vs pop2
vcftools --vcf sim_data_filtered.recode.vcf \
    --weir-fst-pop pop1.txt \
    --weir-fst-pop pop2.txt \
    --out fst_pop1_pop2
```

---

# Resultados

## Simulación de datos RADseq

RADinitio identificó **4,056 loci EcoRI-MseI** en el fragmento de 10 Mb dentro del rango de tamaño objetivo (280–420 pb). Tras eliminar loci en proximidad (<1,000 pb entre sí), se retuvieron **2,788 loci** para la simulación. El modelo coalescente generó **36,684 variantes** distribuidas en las 15 muestras.

## Ensamblaje de loci con ipyrad

| Filtro | Loci eliminados | Loci retenidos |
|--------|----------------|----------------|
| Total prefiltered | — | 2,807 |
| Duplicados eliminados | 48 | 2,759 |
| Max INDELs | 400 | 2,359 |
| Max SNPs por locus | 1 | 2,359 |
| Max heterocigotos compartidos | 67 | 2,306 |
| Min muestras por locus (≥4) | 469 | **1,837** |

La cobertura por muestra fue de ~1,330–1,340 loci retenidos por individuo.

## Filtrado de SNPs

| Parámetro | Valor |
|-----------|-------|
| SNPs antes del filtrado | 1,006 |
| SNPs después del filtrado | **561** |
| Individuos retenidos | 15/15 |
| Criterios aplicados | Sin INDELs, max-missing 0.8, DP 6–100X |

## Diversidad nucleotídica (π)

| Población | Individuos | π promedio |
|-----------|-----------|-----------|
| pop0 | 5 | 0.1964 |
| pop1 | 5 | 0.2386 |
| pop2 | 5 | 0.2281 |

## Heterocigosidad observada

| Individuo | Población | O(HOM) | E(HOM) | N_sitios | F |
|-----------|-----------|--------|--------|----------|---|
| msp_00 | pop0 | 427 | 382.6 | 516 | 0.333 |
| msp_01 | pop0 | 386 | 361.9 | 486 | 0.194 |
| msp_02 | pop0 | 441 | 384.0 | 516 | 0.432 |
| msp_03 | pop0 | 404 | 364.0 | 490 | 0.318 |
| msp_04 | pop0 | 439 | 393.7 | 531 | 0.330 |
| msp_05 | pop1 | 388 | 368.2 | 496 | 0.155 |
| msp_06 | pop1 | 426 | 391.9 | 528 | 0.250 |
| msp_07 | pop1 | 393 | 370.0 | 498 | 0.180 |
| msp_08 | pop1 | 400 | 366.6 | 492 | 0.267 |
| msp_09 | pop1 | 402 | 375.2 | 503 | 0.210 |
| msp_10 | pop2 | 404 | 377.2 | 503 | 0.213 |
| msp_11 | pop2 | 424 | 384.4 | 517 | 0.299 |
| msp_12 | pop2 | 423 | 376.9 | 509 | 0.349 |
| msp_13 | pop2 | 458 | 397.9 | 536 | 0.435 |
| msp_14 | pop2 | 433 | 391.2 | 525 | 0.312 |

> **F** = coeficiente de endogamia de Wright. Valores positivos indican exceso de homocigotos respecto a lo esperado bajo equilibrio Hardy-Weinberg.

## Diferenciación genética (F_ST de Weir y Cockerham)

| Comparación | F_ST medio | F_ST ponderado |
|-------------|-----------|----------------|
| pop0 vs pop1 | 0.084 | 0.152 |
| pop0 vs pop2 | 0.118 | 0.220 |
| pop1 vs pop2 | 0.080 | 0.143 |

> El **F_ST medio** promedia todos los sitios incluyendo los no informativos; el **F_ST ponderado** da mayor peso a los sitios con mayor variación y es el estimador recomendado para comparaciones entre poblaciones.

---

# Discusión

## Loci RAD y filtrado

El pipeline recuperó 1,837 loci de los 2,788 simulados. La mayor fuente de pérdida fue el filtro de mínimo de muestras por locus (`min_samples_locus = 4`), que eliminó 469 loci. Esto es esperable: en datos simulados con un modelo de migración simétrica moderada, algunos loci solo son polimórficos en un subconjunto de poblaciones y no alcanzan el umbral de cobertura mínima en todas las muestras. Este patrón es consistente con lo reportado por Eaton & Overcast (2020) para datasets ddRAD simulados.

Tras el filtrado con VCFtools, se retuvieron 561 SNPs de 1,006 sitios variables (55.8%), principalmente por el criterio de datos faltantes (`max-missing 0.8`). Este porcentaje de retención es razonable para datos RAD de baja cobertura (10X).

## Diversidad nucleotídica

Los valores de π oscilaron entre 0.196 (pop0) y 0.239 (pop1). Bajo el modelo neutral de Jukes-Cantor con Ne = 1,000 y tasa de mutación μ = 7 × 10⁻⁸, la diversidad nucleotídica esperada es:

> θ = 4 × Ne × μ = 4 × 1,000 × 7×10⁻⁸ = **2.8 × 10⁻⁴**

Los valores observados son más altos porque se calcularon solo sobre los sitios polimórficos identificados por el protocolo RAD (561 SNPs), no sobre la totalidad de sitios del genoma. El π calculado sobre sitios variables sobreestima el π genómico real. En un análisis real con pixy sobre un VCF allsites (con sitios invariantes incluidos), el denominador incluiría todos los sitios y el estimador sería insesgado.

## Diferenciación genética

Los valores de F_ST ponderado (0.143–0.220) indican diferenciación moderada entre las tres poblaciones, coherente con el modelo de migración simétrica utilizado (m = 5 × 10⁻⁴). Bajo el modelo de isla de Wright:

> F_ST ≈ 1 / (1 + 4×Ne×m) = 1 / (1 + 4×1000×5×10⁻⁴) = 1 / 3 ≈ **0.333**

Los valores observados (0.143–0.220) son menores que la predicción teórica, lo que puede deberse al tamaño pequeño de muestra (n=5 por población) y al uso de un fragmento reducido del genoma. La comparación pop0 vs pop2 muestra la mayor diferenciación (F_ST = 0.220), mientras que pop1 vs pop2 muestra la menor (F_ST = 0.143), lo que podría reflejar diferencias estocásticas en la historia genealógica de los loci muestreados dado el Ne reducido.

## Limitaciones

- El análisis se realizó sobre un fragmento de 10 Mb del cromosoma 6, lo que limita el número de loci RAD y reduce la precisión de los estimadores poblacionales.
- El tamaño muestral reducido (n=5 por población) introduce alta varianza en los estimadores de π y F_ST.
- El cálculo de π con VCFtools sobre sitios variables únicamente produce estimaciones sesgadas hacia arriba. Para estimaciones insesgadas en datos WGS se recomienda usar pixy con un VCF allsites.

---

# Referencias

- Andrews, K. R., Good, J. M., Miller, M. R., Luikart, G., & Hohenlohe, P. A. (2016). Harnessing the power of RADseq for ecological and evolutionary genomics. *Nature Reviews Genetics*, 17(2), 81–92.
- Baird, N. A., et al. (2008). Rapid SNP discovery and genetic mapping using sequenced RAD markers. *PLOS ONE*, 3(10), e3376.
- Eaton, D. A. R., & Overcast, I. (2020). ipyrad: Interactive assembly and analysis of RADseq datasets. *Bioinformatics*, 36(8), 2592–2594.
- Kelleher, J., Etheridge, A. M., & McVean, G. (2016). Efficient coalescent simulation and genealogical analysis for large sample sizes. *PLOS Computational Biology*, 12(5), e1004842.
- Peterson, B. K., Weber, J. N., Kay, E. H., Fisher, H. S., & Hoekstra, H. E. (2012). Double digest RADseq: An inexpensive method for de novo SNP discovery and genotyping in model and non-model species. *PLOS ONE*, 7(5), e37135.
- Takei, H., et al. (2021). De novo genome assembly of two tomato ancestors, *Solanum pimpinellifolium* and *Solanum lycopersicum* var. *cerasiforme*, by long-read sequencing. *DNA Research*, 28(1), dsaa029.
- Weir, B. S., & Cockerham, C. C. (1984). Estimating F-statistics for the analysis of population structure. *Evolution*, 38(6), 1358–1370.

