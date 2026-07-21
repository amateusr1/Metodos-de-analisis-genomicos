# Ensamblaje de Novo - Plastoma

> Una guía sobre los fundamentos del ensamblaje de Novo y un flujo de trabajo completo con datos de NGS utilizando GetOrganell

---
## Introducción

El ensamblado de novo es un proceso bioinformático que permite reconstruir una secuencia genómica a partir de lecturas obtenidas mediante tecnologías de secuenciación, sin utilizar un genoma de referencia. Este enfoque es especialmente útil cuando se trabaja con especies cuyos genomas no han sido previamente secuenciados o cuando se busca identificar variaciones estructurales que podrían perderse en un ensamblado basado en referencia.

El proceso inicia con el control de calidad de las lecturas, en el que se eliminan adaptadores, bases de baja calidad y secuencias contaminantes. Posteriormente, el software de ensamblado identifica regiones de solapamiento entre las lecturas para construir fragmentos continuos de secuencia denominados contigs. En etapas posteriores, estos contigs pueden organizarse en estructuras de mayor longitud llamadas scaffolds, utilizando información adicional como lecturas pareadas o de larga longitud.

Los algoritmos de ensamblado de novo se basan principalmente en dos estrategias. La primera corresponde al método Overlap-Layout-Consensus (OLC), que compara directamente las lecturas para identificar sus solapamientos, siendo adecuado para lecturas largas. La segunda estrategia utiliza grafos de De Bruijn, en los que las lecturas se dividen en fragmentos más pequeños llamados k-mers. Cada k-mer constituye un nodo del grafo y las conexiones entre ellos permiten reconstruir la secuencia original. Este último enfoque es ampliamente utilizado para datos provenientes de plataformas de secuenciación de lectura corta, como Illumina.

La calidad del ensamblado depende de factores como la profundidad de secuenciación, la longitud de las lecturas, la presencia de regiones repetitivas, el contenido de errores de secuenciación y la complejidad del genoma. Para evaluar el resultado se emplean métricas como el número de contigs, la longitud total ensamblada, el valor N50, el porcentaje de cobertura y la completitud del genoma obtenido.

Los plastidios son orgánulos presentes en plantas y algas que contienen un genoma propio, generalmente de estructura circular, altamente conservado y con un tamaño aproximado entre 120 y 170 kilobases en plantas terrestres. El genoma plastidial suele presentar una organización característica formada por una región larga de copia única (LSC, Large Single Copy), una región corta de copia única (SSC, Small Single Copy) y dos regiones repetidas invertidas (IR, Inverted Repeats).

El ensamblado de genomas plastidiales puede realizarse a partir de datos de secuenciación del ADN total debido a que las células vegetales contienen numerosas copias del genoma del cloroplasto, lo que genera una alta cobertura de estas secuencias. En consecuencia, es posible reconstruir el genoma plastidial sin necesidad de aislar previamente los cloroplastos.

Existen diferentes estrategias para el ensamblado de genomas plastidiales. Una consiste en extraer previamente las lecturas correspondientes al plastidio mediante alineamiento contra un genoma de referencia cercano y posteriormente ensamblarlas. Otra alternativa es realizar un ensamblado de novo utilizando únicamente la información contenida en las lecturas y, posteriormente, identificar los contigs pertenecientes al genoma plastidial. Herramientas especializadas como GetOrganelle, NOVOPlasty y Fast-Plast han sido desarrolladas para facilitar este proceso mediante algoritmos optimizados para genomas organelares.

Una vez obtenido el ensamblado, es necesario verificar que el genoma sea completo y presente la estructura cuadripartita esperada. Posteriormente, se realiza la anotación de genes codificantes de proteínas, ARN ribosomales (rRNA) y ARN de transferencia (tRNA), así como la identificación de las regiones LSC, SSC e IR. La calidad del ensamblado también puede confirmarse mediante el análisis de la cobertura de secuenciación y la comparación con genomas plastidiales de especies relacionadas.

Los genomas plastidiales constituyen una fuente importante de información para estudios de evolución, filogenia, sistemática, biogeografía y genética de poblaciones, debido a su relativa conservación estructural, su herencia predominantemente materna en la mayoría de las angiospermas y su baja tasa de recombinación.
