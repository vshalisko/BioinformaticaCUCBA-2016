# Datos genéticos utilizados en bioinformática 

Referencias recomendadas para esta Unidad:

* [Next Generation Sequencing (NGS)/Pre-Processing - Wikibooks, Open Books for an Open World (2014)](http://en.wikibooks.org/wiki/Next_Generation_Sequencing_(NGS)/Pre-processing#Sequence_Quality)

	

## Datos crudos 	

### ¿Qué son los datos *crudos*?

Son las secuencias tal cual salen de la plataforma de secuenciación (Illumina, IonTorrent, PacBio, entre otros). Es decir los **reads** (lecturas).  


Los datos crudos (sin importar la plataforma y el método de laboratorio utilizado) conllevan cierto nivel de basura, es decir:

* *reads* de baja calidad que deben ser descartados por completo o en parte (*trimmed*) antes de proceder a los análisis biológicos de los datos. 

* Secuencias sobrerepresentadas, por ejemplo dímeros de adaptadores.

La longitud y distribución de la calidad de los *reads* varían de plataforma a plataforma, pero también el tipo de error más común:

* Illumina: errores puntuales en la asignación de un nucleótido
* IonTorrent y símiles: se les complica determinar el número correcto de nucleótidos en cadenas como AAAAA.

Dado que los datos crudos son muy pesados (espacio de disco) y que buena parte de los datos crudos son basura, en general los datos crudos a este nivel no se guardan en repositorios públicos (e.g. SRA) hasta que hayan pasado por los filtros del pre-procesamiento.

Como veremos más adelante, los filtros de pre-procesamiento ayudan a identificar las buenas secuencias de la basura a partir de su calidad asociada.


## Información en los archivos FASTQ 		
### Representación de secuencias

En cómputo las secuencias de ADN son una *string* (cadena) de caracteres. 

* Secuencias genómicas

{A,C,G,T}+

* Secuencias mRNA

{A,C,G,U}+



##### Secuencias simples: FASTA/Pearson Format

Línea 1: información de la secuencia

Línea 2: la secuencia.

```

>gi|365266830|gb|JF701598.1| Pinus densiflora var. densiflora voucher Pf0855 trnS-trnG intergenic spacer, partial sequence; chloroplast
GCAGAAAAAATCAGCAGTCATACAGTGCTTGACCTAATTTGATAGCTAGAATACCTGCTGTAAAAGCAAG
AAAAAAAGCTATCAAAAATTTAAGCTCTACCATATCTTCATTCCCTCCTCAATGAGTTTGATTAAATGCG
TTACATGGATTAGTCCATTTATTTCTCTCCAATATCAAATTTTATTATCTAGATATTGAAGGGTTCTCTA
TCTATTTTATTATTATTGTAACGCTATCAGTTGCTCAAGGCCATAGGTTCCTGATCGAAACTACACCAAT
GGGTAGGAGTCCGAAGAAGACAAAATAGAAGAAAAGTGATTGATCCCGACAACATTTTATTCATACATTC
AGTCGATGGAGGGTGAAAGAAAACCAAATGGATCTAGAAGTTATTGCGCAGCTCACTGTTCTGACTCTGA
TGGTTGTATCGGGCCCTTTAGTTATTGTTTTATCAGCAATTCGCAAAGGTAATCTATAATTACAATGAGC
CATCTCCGGAGATGGCTCATTGTAATGATGAAAACGAGGTAATGATTGATATAAACTTTCAATAGAGGTT
GATTGATAACTCCTCATCTTCCTATTGGTTGGACAAAAGATCGATCCA

```


##### Fastq: formato para secuencias de secuenciación de siguiente generación

Secuencia fasta + detalles calidad de la información (la Q es de Quality).

* Línea 1: Encabezado (*Header*): comienza con @. Sigue el identificador (*identifier*). Si son datos crudos contiene info del secuenciador que identifica a esta secuencia y el read pair (/1 o /2), si son datos ya procesados en SRA contiene una descripción de la secuencia.

* Línea 2: la secuencia. 

* Línea 3: Comienza con +. Puede ser sólo el símbolo + o repetir la info del Header. 

* Línea 4: Información de la calidad de secuenciación de cada base. Cada letra o símbolo representa a una base de la secuencia codificado en formato [ASCII](http://ascii.cl/). 


La info de calidad se codifica en ASCII porque esto permite en 1 sólo caracter codificar un valor de calidad. Por lo tanto la línea 2 y la 4 tienen el mismo número de caracteres. 
 

Ordenados de menor a mayor estos son los caracteres ASCII usados para representar calidad en FASTQ:

```
!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
```
Dependiendo del tipo y versión de plataforma de secuenciación se toman diferentes caracteres (pero sin desordenarlos):


![ascii_fastqplataformas.png](ascii_fastqplataformas.png)

(Tomé la imágen de [aquí](http://en.wikibooks.org/wiki/Next_Generation_Sequencing_(NGS)/Pre-processing#Sequence_Quality)
)

Pero en todos los casos el **valor máximo de calidad es = ~40** y los valores **< 20 se consideran bajos**.


**Ojo:**
 @ y + están dentro de los caracteres ASCII utilizados para codificar la calidad, lo que hace que usar `grep` en archivos FASTQ pueda complicarse, ojo.

**Ejemplos:**

Ejemplo de datos FASTQ recién salidos de Illumina:

```
@HWI-ST999:102:D1N6AACXX:1:1101:1235:1936 1:N:0:
ATGTCTCCTGGACCCCTCTGTGCCCAAGCTCCTCATGCATCCTCCTCAGCAACTTGTCCTGTAGCTGAGGCTCACTGACTACCAGCTGCAG
+
1:DAADDDF<B<AGF=FGIEHCCD9DG=1E9?D>CF@HHG??B<GEBGHCG;;CDB8==C@@>>GII@@5?A?@B>CEDCFCC:;?CCCAC
```
Y uno más:

```
@OBIWAN:24:D1KUMACXX:3:1112:9698:62774 1:N:0:
TAATATGGCTAATGCCCTAATCTTAGTGTGCCCAACCCACTTACTAACAAATAACTAACATTAAGATCGGAAGAGCACACGTCTGAACTCAGTCACTGACC
+
CCCFFFFFHHHHHIJJJJJJJJJJJJIIHHIJJJJJJJJJJJJJJJJJJJJIJJJJJJIJJJJIJJJJJJJHHHHFDFFEDEDDDDDDDDDDDDDDDDDDC

```
¿Quieres saber cuáles son las partes del Header? [Clic aquí](https://en.wikipedia.org/wiki/FASTQ_format#Illumina_sequence_identifiers). Y sí, el último ejemplo es real y por lo tanto hay un Illumina HiSeq2000 que se llama Obiwan :)

 Ejemplo de datos FASTQ del SRA:

```
@SRR001666.1 071112_SLXA-EAS1_s_7:5:1:817:345 length=36
GGGTGATGGCCGCTGCCGATGGCGTCAAATCCCACC
+SRR001666.1 071112_SLXA-EAS1_s_7:5:1:817:345 length=36
IIIIIIIIIIIIIIIIIIIIIIIIIIIIII9IG9IC
```


Los datos FASTQ típicamente están comprimidos en formato 
`gzip` (.gz) o `tar` (.tar.gz o .tgz).


Los datos crudos deben filtrarse y editarse para quedarnos con los **datos limpios**, que son los que se analizarán para resolver preguntas biológicas. A esto se le conoce como [Pre-procesamiento](2_Pre-procesamiento.md) y lo veremos en su propia sección. 


## Datos procesados

Son los datos *output* de procesar los datos crudos (ya limpios) con un programa bioinformático para darles significado biológico, como son (dependiendo de la naturaleza de los datos): 

* Ensamblado *de novo*
* Mapeo a genoma de referencia
* Genotipificación

Los datos procesados en dicho formato específico serán a su vez el *input* de nuevos análisis, por ejemplo de genómica de poblaciones, genómica comparativa, filogenética, niveles de expresión génica o diferenciación de comunidades. 

### Formatos de datos procesados

Dependiendo del **tipo de procesamiento** y del **software** que se utilizó los datos procesados podrán pasar del formato .fastq (y considerarse meramente *reads*) a un **formato específico** según si se trata de alineamientos, genotipificaciones, etc. 

Muchos formatos están asociados a un programa en particular, aunque los más usados son relativamente convencionales y ya pueden ser utilizados por otros programas y herramientas.

Otros programas tienen sus propios formatos y hay que transformarlos manualmente (bueno, con la línea de comando) para analizarlos con otro software (lo cual puede ser doloroso).

Por lo tanto hay decenas de formatos. Algunos listados ejemplo:

[Utilizados por la UCSC](https://genome.ucsc.edu/FAQ/FAQformat.html) 

[Utilizados por the Ensembl Project](http://www.ensembl.org/info/website/upload/index.html#formats)

[Recomendados por the Broad Institute según tipo de datos](https://www.broadinstitute.org/igv/RecommendedFileFormats)

[Formatos de OBITools](http://pythonhosted.org/OBITools/welcome.html) y [QUIME](http://qiime.org/documentation/index.html) (metabarcoding).

[Plink porque es bastante estándar](https://www.cog-genomics.org/plink2/)

y [VCF también](http://www.1000genomes.org/wiki/Analysis/variant-call-format)

A continuación explicaremos algunos de los formatos más comunes. 

#### BED 
"Browser Extensible Data" [Ref](https://genome.ucsc.edu/FAQ/FAQformat.html#format1)

Se utiliza en genomas anotados, determina la localización precisa y estructura (intervalos, listas de intervalos e info biológica asociada a ellos) de características genómicas (genes, promotores, sitios de inicio/fin de la traducción, etc) a lo largo del genoma. 

Por ejemplo: 
coordenadas de inicio/fin de exon/intron

Contenido del Formato Básico (obligatorios):

`#Chr start end`

Contenido del Formato Extendendido (opcionales)

`#Chr start end name score strand thick_start thick_end rgb blockCount blockSizes blockStarts`

Ejemplo:

```
track name=pairedReads description="Clone Paired Reads" useScore=1
chr22 1000 5000 cloneA 960 + 1000 5000 0 2 567,488, 0,3512
chr22 2000 6000 cloneB 900 - 2000 6000 0 2 433,399, 0,3601
```

Los formatos BED se pueden ver en Genome Browsers, [ejemplo](https://genome.ucsc.edu/cgi-bin/hgTracks?db=hg38&lastVirtModeType=default&lastVirtModeExtraState=&virtModeType=default&virtMode=0&nonVirtPosition=&position=chr7%3A127471196-127495720&hgsid=480754503_SpAYN0AAYEaAqBRCoWi1iwwRkgpK)

**Ojo: hay diferentes formas de contar inicio/fin de coordenadas**

* 0-based: cuenta espacios (entre letra y letra) 
* 1-based: cuenta las bases


#### Formato SAM/BAM
"Sequence Alignment Map", su versión binaria (comprimida) BAM.
[Ref](https://samtools.github.io/hts-specs/SAMv1.pdf)

Formato para representar un alineamiento de NGS seq.

Programa asociado: [Samtools](https://github.com/samtools/samtools)

Alineamiento: Mapear las letras (bases) de dos o más secuencias, permitiendo algunos espaciadores (indels). 

Variaciones dentro de un alineamiento:
Indels
Substituciones

El ADN se alinea de forma continua al genoma

Pero el mRNA puede tener un alineamiento dividido (spliced aligment) -> <- y forman un continuo

Contenido del formato SAM:

Header: líneas que empiezan con `@` y dan información del alineamiento, la longitud de cada cromosoma, programa con el que se hizo, etc.

Alineamiento: una línea por cada alineamiento.

Contenido de las columnas: 
```
Read id
FLAG
Chr
Start
Mapping quality
CIGAR (aligment)
MateChr
MateStart
MateDist
QuerySeq
QueryBaseQuals
AlignmentScore
Edot-distance-to-reference
Number-of-hits
Strand
Hit-index
```

Ejemplo:

Un alineamiento así:

```
Coor     12345678901234  5678901234567890123456789012345
ref      AGCATGTTAGATAA**GATAGCTGTGCTAGTAGGCAGTCAGCGCCAT
+r001/1        TTAGATAAAGGATA*CTG
+r002         aaaAGATAA*GGATA
+r003       gcctaAGCTAA
+r004                     ATAGCT..............TCAGC
-r003                            ttagctTAGGC
-r001/2                                        CAGCGGCAT
```

En SAM se codifica así:

```
@HD VN:1.5 SO:coordinate
@SQ SN:ref LN:45
r001   99 ref  7 30 8M2I4M1D3M = 37  39 TTAGATAAAGGATACTG *
r002    0 ref  9 30 3S6M1P1I4M *  0   0 AAAAGATAAGGATA    *
r003    0 ref  9 30 5S6M       *  0   0 GCCTAAGCTAA       * SA:Z:ref,29,-,6H5M,17,0;
r004    0 ref 16 30 6M14N5M    *  0   0 ATAGCTTCAGC       *
r003 2064 ref 29 17 6H5M       *  0   0 TAGGC             * SA:Z:ref,9,+,5S6M,30,1;
r001  147 ref 37 30 9M         =  7 -39 CAGCGGCAT         * NM:i:1
```

#### VCF
"Variant Call Format"
[Ref](http://samtools.github.io/hts-specs/VCFv4.2.pdf)

Formato para representar una posición en el genoma (posiblemente con variantes) y su información asociada. También puede contener información de genotipos de varias muestras para cada posición. 

Programa asociado: [VCFtools](https://vcftools.github.io/index.html) y [BCFtools](https://github.com/samtools/bcftools)

Ejemplo:

```
##fileformat=VCFv4.2
##fileDate=20090805
##source=myImputationProgramV3.1
##reference=file:///seq/references/1000GenomesPilot-NCBI36.fasta
##contig=<ID=20,length=62435964,assembly=B36,md5=f126cdf8a6e0c7f379d618ff66beb2da,species="Homo sapiens",taxonomy=x>
##phasing=partial
##INFO=<ID=NS,Number=1,Type=Integer,Description="Number of Samples With Data">
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
##INFO=<ID=AF,Number=A,Type=Float,Description="Allele Frequency">
##INFO=<ID=AA,Number=1,Type=String,Description="Ancestral Allele">
##INFO=<ID=DB,Number=0,Type=Flag,Description="dbSNP membership, build 129">
##INFO=<ID=H2,Number=0,Type=Flag,Description="HapMap2 membership">
##FILTER=<ID=q10,Description="Quality below 10">
##FILTER=<ID=s50,Description="Less than 50% of samples have data">
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
##FORMAT=<ID=GQ,Number=1,Type=Integer,Description="Genotype Quality">
##FORMAT=<ID=DP,Number=1,Type=Integer,Description="Read Depth">
##FORMAT=<ID=HQ,Number=2,Type=Integer,Description="Haplotype Quality">
#CHROM POS     ID        REF    ALT     QUAL FILTER INFO                              FORMAT      NA00001        NA00002        NA00003
20     14370   rs6054257 G      A       29   PASS   NS=3;DP=14;AF=0.5;DB;H2           GT:GQ:DP:HQ 0|0:48:1:51,51 1|0:48:8:51,51 1/1:43:5:.,.
20     17330   .         T      A       3    q10    NS=3;DP=11;AF=0.017               GT:GQ:DP:HQ 0|0:49:3:58,50 0|1:3:5:65,3   0/0:41:3
20     1110696 rs6040355 A      G,T     67   PASS   NS=2;DP=10;AF=0.333,0.667;AA=T;DB GT:GQ:DP:HQ 1|2:21:6:23,27 2|1:2:0:18,2   2/2:35:4
20     1230237 .         T      .       47   PASS   NS=3;DP=13;AA=T                   GT:GQ:DP:HQ 0|0:54:7:56,60 0|0:48:4:51,51 0/0:61:2
20     1234567 microsat1 GTC    G,GTCT  50   PASS   NS=3;DP=9;AA=G                    GT:GQ:DP    0/1:35:4       0/2:17:2       1/1:40:
```



#### Plink

Sirve para analizar SNPs de varias (miles) de muestras a la vez. [Ref](https://www.cog-genomics.org/plink2/)

Programa asociado: [Plink](http://pngu.mgh.harvard.edu/~purcell/plink/) y [Plink2]((https://www.cog-genomics.org/plink2/))

En realidad hay varios [tipos de formato plink](https://www.cog-genomics.org/plink2/formats), y normalmente no son uno sino **varios archivos**. 

La manera de manejar los formatos cambió un poco entre las versiones <1.9 y 1.9. Siguen siendo compatibles, pero aguas.

**Plink text (`ped`)**

Consta de min 2 archivos: 

`.ped` que contiene los SNPs
```
plink.ped:
  1 1 0 0 1  0  G G  2 2  C C
  1 2 0 0 1  0  A A  0 0  A C
  1 3 1 2 1  2  0 0  1 2  A C
  2 1 0 0 1  0  A A  2 2  0 0
  2 2 0 0 1  2  A A  2 2  0 0
  2 3 1 2 1  2  A A  2 2  A A
```

`.map` que contiene la localización de esos SNPs

```
plink.map:
  1 snp1 0 1
  1 snp2 0 2
  1 snp3 0 3
```

**Plink 1 binario (`bed`)**
Es una versión binaria de lo anterior. Consta de 3 archivos:

`.bed` (PLINK binary biallelic genotype table). Ojo, no confundir con el `BED` que vimos arriba.

`.bim` que contiene las bases originales de cada SNP

```
plink.bim  
  1  snp1  0  1  G  A
  1  snp2  0  2  1  2
  1  snp3  0  3  A  C
```

`.fam` (PLINK sample information file) que contiene info de las muestras. Una línea por muestra con la siguiente info:

    Family ID ('FID')
    Within-family ID ('IID'; cannot be '0')
    Within-family ID of father ('0' if father isn't in dataset)
    Within-family ID of mother ('0' if mother isn't in dataset)
    Sex code ('1' = male, '2' = female, '0' = unknown)
    Phenotype value ('1' = control, '2' = case, '-9'/'0'/non-numeric = missing data if case/control)

Plink está pensado para datos humanos (de ahí lo de familia, madre, sexo, etc), pero es posible tener datos en formato plink sin ese tipo de información. 

Muchos programas de genética de poblaciones y R utilizan plink para correr. 

**Plink `raw`** (additive + dominant component file)

Es un formato producido por Plink para realizar análisis en R (pero no en Plink).

Contenido:

Header line y luego una línea por muestra con: 

```
V+6 (for '--recode A') or 2V+6 (for '--recode AD') fields, where V is the number of variants. 
FID	Family ID
IID	Within-family ID
PAT	Paternal within-family ID
MAT	Maternal within-family ID
SEX	Sex (1 = male, 2 = female, 0 = unknown)
PHENOTYPE	Main phenotype value
```

Ejemplo:

```
FID	IID	PAT	MAT	SEX	PHENOTYPE	abph1.15_G	ae1.8_A	an1.3_A	ba1.5_G	ba1.7_A	csu1138.4_A	csu1171.2_A	Fea2.2_A	fea2.3_G	MZB00125.2_A	pbf1.3_G1	maiz_3	0	0	0	-9	2	0	0	2	0	0	2	0	0	0	02	maiz_68	0	0	0	-9	0	0	2	1	0	0	0	0	0	0	03	maiz_91	0	0	0	-9	2	0	0	0	0	0	2	0	0	1	0
```



#### Documentos de texto 

Archivos csv con las secuencias e información asociada. Programas como Stacks, pyRAD, OBITools y muchos otros los manejan. 

















