# Introducción a Linux 

Este archivo reúne una serie de comandos básicos y útiles de Linux orientados al trabajo en clústeres. La idea es tener una guía rápida de consulta para tareas frecuentes como manejo de archivos, navegación entre directorios, ejecución y monitoreo de trabajos, administración de ambientes, transferencia de datos y uso de software bioinformático. Muchos de estos comandos son esenciales para trabajar de forma eficiente en análisis computacionales y pipelines dentro de ambientes HPC (High Performance Computing), especialmente en proyectos de bioinformática y análisis de datos.

| Función                        | Scripts                              |
|--------------------------------|--------------------------------------|
|  Borrar archivos   | ```rm```|
|  Borrar directorios  | ```rm -rf```|
|  Nodos disponibles para someter Jobs  | ```sinfo -p *```                          | 
|  Someter Jobs  | ```sbatch```                         |
|  Ver mis Jobs sometidos  | ```squeue -u amateusr```                          |
|  Monitorear el progreso del output en tiempo real   | ```tail -f *.out```                          |
|  Ver lista de modulos | ```module avail```                          |
|  Crear un nuevo directorio  | ```mkdir```                          |
|  Ver espacio disponible en el directorio  | ```df -h *```|
|  Ver como crece la carpeta   | ```du -sh```|
|  Ver tamaño de archivos   | ```ls -lh```|
|  Visualizar archivos   | ```cat```|
|  Editar archivos   | ```nano - Guarda con Ctrl+O, Enter, Ctrl+X y lanza```|
|  Cancelar un Job   | ```scancel```|
|  Matar, finalizar un proceso   | ```Control + C```|
|  Mover o renombrar   | ```mv```|
|  Copiar archivos o directorios   | ```cp```|
|  Moverse entre directorios   | ```cd ../```|
|  Listar objetos del directorio   | ```ls```|
|  Crear archivo vacío .txt   | ```touch```|
|  Otorgar permisos de ejecución a un archivo .sh  | ```chmod +x nombre_archivo.sh```|
|  Verificar que el .fastq.gz este bien  |  ```zcat nombre* | head -12``` |

