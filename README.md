# Práctica SARS-CoV-2
En este repositorio encontrarás las instrucciones para la práctica del genoma del virus SARS-CoV-2

## Programas requeridos:

### SO Ubuntu o Mac
Esta práctica está diseñada para Linux o Mac. 
En caso de tener Windows 10 se puede optar por la instalación de la terminal Ubuntu 18.04.
### SPAdes 3.15.2
Las instrucciones de instalación y requerimientos se encuentran aquí:
https://github.com/ablab/spades
También se puede instalar mediante conda:
https://bioconda.github.io/recipes/spades/README.html
Asegurarse de que la versión incluya coronaSPAdes
### Bowtie2
Las instrucciones de instalación se encuentran aquí:
https://github.com/BenLangmead/bowtie2
### FastQC
https://www.bioinformatics.babraham.ac.uk/projects/fastqc/

## Datos requeridos
Para la práctica llevaremos necesitaremos descargar:

#### Genoma del virus SARS-CoV-2:
Se encuentra en GenBank con el nombre de acceso: MT318827 (Pfefferle, S. et al., 2020). 
Igualmente se puede utilizar algún otro genoma secuenciado completo.
Para descargar este genoma se puede hacer mediante el uso de la herramienta entrez.
Para instalarla:

  apt install ncbi-entrez-direct

Una vez instalada, para descargar el genoma en formato fasta:

  efetch -db nucleotide -id MT318827 -mode text -format fasta > SARS_CoV_2_genome.fasta

#### Datos de secuenciación 
Por otro lado, descargaremos reads procedentes de una secuenciación con el acceso PRJNA624231 (BioProject), SRR11517432 (SRA), y SAMN14572083 (BioSample) (Pfefferle, S. et al., 2020). Estos serán los que ensamblaremos.
Para descargarlos se puede usar el SRA toolkit https://www.ncbi.nlm.nih.gov/sra/docs/sradownload/:

  prefetch SRR11517432
  fastq-dump --outdir reads --skip-technical -I -W --split-files SRR11517432/SRR11517432.sra

## Parte 1
### ¿De donde proceden los datos?
Los datos provienen de una muestra de hisopado orofaríngeo de un paciente con COVID-19 que se encontraba en Hamburgo, al norte de Alemania.
Se recomienda leer la publicación (Pfefferle, S. et al., 2020).
Estos datos de RNAseq, se obtuvieron mediante el empleo de la plataforma Illumina NextSeq.

## Parte 2
### Visualización de datos
Como recordarán el primer paso es visualizar los reads y sus calidades.
Para ello emplearemos la herramienta fastqc.
Recuerden que podemos usar el ambiente gráfico de fastqc o con línea de comandos teclear:
  
  fastqc reads/*

para realizar todos los análisis.
Y luego visualizar los html generados.

## Parte 3
### Filtrado/limpieza de reads




Pfefferle, S., Huang, J., Nörz, D., Indenbirken, D., Lütgehetmann, M., Oestereich, L., ... & Fischer, N. (2020). Complete genome sequence of a SARS-CoV-2 strain isolated in Northern Germany. Microbiology resource announcements, 9(23). doi:10.1128/MRA.00520-20
