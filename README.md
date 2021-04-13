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
### Trimmomatic
http://www.usadellab.org/cms/?page=trimmomatic
Se puede instalar mediante conda:
https://anaconda.org/bioconda/trimmomatic
### Samtools
https://www.howtoinstall.me/ubuntu/18-04/samtools/
### n50 script
  *conda install -y -c bioconda n50*

## Datos requeridos
Para la práctica necesitaremos descargar:

#### Genoma del virus SARS-CoV-2:
Se encuentra en GenBank con el nombre de acceso: MT318827 (Pfefferle, S. et al., 2020). 
Igualmente se puede utilizar algún otro genoma secuenciado completo.
Para descargar este genoma se puede hacer mediante el uso de la herramienta entrez.
Para instalarla:

  *apt install ncbi-entrez-direct

Una vez instalada, para descargar el genoma en formato fasta:

  *efetch -db nucleotide -id MT318827 -mode text -format fasta > SARS_CoV_2_genome.fasta

#### Datos de secuenciación 
Por otro lado, descargaremos reads procedentes de una secuenciación con el acceso PRJNA624231 (BioProject), SRR11517432 (SRA), y SAMN14572083 (BioSample) (Pfefferle, S. et al., 2020). Estos serán los que ensamblaremos.
Para descargarlos se puede usar el SRA toolkit https://www.ncbi.nlm.nih.gov/sra/docs/sradownload/:

  *prefetch SRR11517432
  
  *fastq-dump --outdir reads --skip-technical -I -W --split-files SRR11517432/SRR11517432.sra

## Parte 1
### ¿De dónde proceden los datos?
Los datos provienen de una muestra de hisopado orofaríngeo de un paciente con COVID-19 que se encontraba en Hamburgo, al norte de Alemania.
Se recomienda leer la publicación (Pfefferle, S. et al., 2020).
Estos datos de RNAseq, se obtuvieron mediante el empleo de la plataforma Illumina NextSeq.

## Parte 2
### Visualización de datos
Como recordarán el primer paso es visualizar los reads y sus calidades.
Para ello emplearemos la herramienta fastqc.
Recuerden que podemos usar el ambiente gráfico de fastqc o con línea de comandos teclear:
  
  *fastqc reads/*

para realizar todos los análisis.
Y luego visualizar los html generados, y de este modo decidir como filtraremos nuestros reads.

## Parte 3
### Filtrado/limpieza de reads
Nos debemos asegurar que no haya secuencias de adaptadores y tengamos una calidad adecuada.
Para limpiar usaremos trimmomatic:

  *trimmomatic PE -phred33 SRR11517432_1.fastq SRR11517432_2.fastq SR11517432_1_pf.fastq SR11517432_1_uf.fastq SR11517432_2_pf.fastq SR1151
7432_2_uf.fastq LEADING:7 TRAILING:3 SLIDINGWINDOW:4:16 MINLEN:35 AVGQUAL:30

Como se puede observar estamos quitando los primeros 7 nucleótidos del inicio de los reads,
los últimos tres, y revisamos que la calidad de los reads sea mayor a 16 con ventanas de 4 nucleótidos,
además sólo nos quedamos con los reads de longitud mayor a 35 tras el filtrado y que conserven una calidad media mayor a 30.

Al finalizar procedemos a visualizar los datos mediante el uso de fastqc

  *fastqc SR11517432_1_pf.fastq
  
  *fastqc SR11517432_2_pf.fastq
  
¿Parece que las calidades mejoraron?
¿Cuántos reads se conservaron?

Ahora cambiaremos los nombres de los headers para que sean los mismos de forward y reverse:

  *awk '{print (NR%4 == 1) ? "@" ++i : $0}' SR11517432_1_pf.fastq > SR11517432_1_pf_c.fastq
  
  *awk '{print (NR%4 == 1) ? "@" ++i : $0}' SR11517432_2_pf.fastq > SR11517432_2_pf_c.fastq


## Parte 4
### Mapeo de reads contra el genoma de referencia

Tenemos datos de un metatranscriptoma del cual las secuencias polyA fueron eliminadas, sin embargo, el resto siguen estando presentes, es decir,
todo el rna expresado por la comunidad bacteriana y todos los virus con genoma compuesto por RNA. Por ello, es necesario, para facilitar el ensamblado,
obtener todos los posibles reads que probablemente son del virus SARS-CoV-2.

Lo que haremos será mapear los reads al genoma que descargamos. Entonces utilizaremos la herramienta Bowtie 2
Primero crearemos los índices con el comando:

  *bowtie2-build SARS_CoV_2_genome.fasta SARS_CoV_2_genome

¿Por qué crear índices? La respuesta es simple, como se realizarán alineamientos se requerirá mucha memoria, entonces los desarrolladores
de Bowtie integraron el indexado Burrows-Wheeler que funciona para comprimir la información de las secuencias fasta.

Ya que se generaron los índices mapeamos nuestros reads filtrados:

  *bowtie2 -x ../SARS_CoV_2_genome -1 SR11517432_1_pf_c.fastq -2 SR11517432_2_pf_c.fastq -S SARS_mapping.sam

Extraemos los reads mapeados mediante el uso de las herramientas samtools:
  
  *samtools view -b -F 4 SARS_mapping.sam > SARS_mapped.bam

El formato sam contiene las secuencias, así como ciertas anotaciones llamadas banderas que corresponden a diferentes características de las secuencias.
En este caso la bandera 4 describe si se alineó contra el genoma de referencia o no. Con "-F" señalamos que queremos los reads mapeados y con "-b" que queremos
un archivo de salida en formato bam, cuyo formato es una versión comprimida del sam.

Posteriormente ordenamos los reads:

  *samtools sort -n SARS_mapped.bam > SARS_mapped_sorted.bam

Y con este comando pasamos a formato fastq los reads mapeados.

  *bamToFastq -i SARS_mapped.bam -fq SARS_1.fastq -fq2 SARS_2.fastq
  
Finalmente analizamos con fastqc las secuencias:

  *fastqc SARS_1.fastq
  
  *fastqc SARS_2.fastq
  
¿De qué tamaño es el genoma del virus? 
¿Tendremos suficiente profundidad de secuenciación?


## Parte 5
### Ensamblaje de novo

Existe en la actualidad un pipeline en específico de spades para ensamblar genomas virales.
Este tiene algunas modificaciones que aprovecha el conocimiento sobre las estructuras del genoma viral para mejorar el ensamblaje.
Para saber más detalles se sugiere consultar la publicación del software 
El comando es el siguiente:

 *coronaspades.py -1 SARS_1.fastq -2 SARS_2.fastq -o SARS_ensamble_spades

¿Cuántos contigs fueron generados?
¿De qué longitud cada uno?

Comprueba esto con el comando:

  *n50 contigs.fasta

Pfefferle, S., Huang, J., Nörz, D., Indenbirken, D., Lütgehetmann, M., Oestereich, L., ... & Fischer, N. (2020). Complete genome sequence of a SARS-CoV-2 strain isolated in Northern Germany. Microbiology resource announcements, 9(23). DOI: 10.1128/MRA.00520-20

Meleshko, D., & Korobeynikov, A. (2020). coronaSPAdes: from biosynthetic gene clusters to coronaviral assemblies. bioRxiv. DOI: 10.1101/2020.07.28.224584


