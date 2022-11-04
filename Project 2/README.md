# Project 2. “Why did I get the flu?”. Deep sequencing, error control, p-value, viral evolution.
Создаем рабочую директорию **Project2**.

## Подготовка данных
1. Создаем директорию для сырых данных **rowData** в **Project2** и переходим в нее.
2. Скачиваем данные секвенирования, полученные от болльного `wget http://ftp.sra.ebi.ac.uk/vol1/fastq/SRR170/001/SRR1705851/SRR1705851.fastq.gz`, `wgetftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR170/008/SRR1705858/SRR1705858.fastq.gz`, `wgetftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR170/009/SRR1705859/SRR1705859.fastq.gz`, `wgetftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR170/000/SRR1705860/SRR1705860.fastq.gz` и разархивируем файл `gunzip SRR1705851.fastq.gz`, `gunzip SRR1705858.fastq.gz`, `gunzip SRR1705859.fastq.gz`, `gunzip SRR1705860.fastq.gz`.
3. Скачиваем референсную последовательность гена вируса с https://www.ncbi.nlm.nih.gov/nuccore/KF848938.1?report=fasta.
4. Проверим сколько ридов в скачанных файлах `wc -l SRR1705851.fastq`, `wc -l SRR1705858.fastq`, `wc -l SRR1705859.fastq`, `wc -l SRR1705860.fastq`. Запросы возвращают числа 1433060, 1026344, 933308, 999856 на каждый рид уходит 4 строки, следовательно, всего 358265, 256586, 233327, 249964 ридов.

## Выравнивание ридов на референс
1. Индексируем референсную последовательность с помощью программы **bwa** `bwa index sequence.fasta`.

<details>
<summary>Output:</summary>
 
```
[bwa_index] Pack FASTA... 0.00 sec
[bwa_index] Construct BWT for the packed sequence...
[bwa_index] 0.00 seconds elapse.
[bwa_index] Update BWT... 0.00 sec
[bwa_index] Pack forward-only FASTA... 0.00 sec
[bwa_index] Construct SA from BWT and Occ... 0.00 sec
[main] Version: 0.7.17-r1188
[main] CMD: bwa index sequence.fasta
[main] Real time: 0.027 sec; CPU: 0.005 sec
```
 </details>
 
2. Выполняем выравнивание ридов с помощью bwa `bwa mem sequence.fasta SRR1705851.fastq > ../processedData/alignment_SRR1705851.sam`, `bwa mem sequence.fasta SRR1705858.fastq > ../processedData/alignment_SRR1705858.sam`, `bwa mem sequence.fasta SRR1705859.fastq > ../processedData/alignment_SRR1705859.sam`, `bwa mem sequence.fasta SRR1705860.fastq > ../processedData/alignment_SRR1705860.sam`.


3. Конвертируем .sam файл в .bam файл. Для этого перейдем в директорию **processedData** и используем программу **samtools** `samtools view -S -b alignment_SRR1705851.sam > alignment_SRR1705851.bam`, `samtools view -S -b alignment_SRR1705858.sam > alignment_SRR1705858.bam`, `samtools view -S -b alignment_SRR1705859.sam > alignment_SRR1705859.bam`, `samtools view -S -b alignment_SRR1705860.sam > alignment_SRR1705860.bam`.
4. Проверим, сколько ридрв было выравнено `samtools flagstat alignment_SRR1705851.bam`, `samtools flagstat alignment_SRR1705858.bam`, `samtools flagstat alignment_SRR1705859.bam`, `samtools flagstat alignment_SRR1705860.bam`.

<details>
<summary>Output:</summary>
 
```
361349 + 0 in total (QC-passed reads + QC-failed reads)
358265 + 0 primary
0 + 0 secondary
3084 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
361116 + 0 mapped (99.94% : N/A)
358032 + 0 primary mapped (99.93% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```
```
256744 + 0 in total (QC-passed reads + QC-failed reads)
256586 + 0 primary
0 + 0 secondary
158 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
256658 + 0 mapped (99.97% : N/A)
256500 + 0 primary mapped (99.97% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```
```
233451 + 0 in total (QC-passed reads + QC-failed reads)
233327 + 0 primary
0 + 0 secondary
124 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
233375 + 0 mapped (99.97% : N/A)
233251 + 0 primary mapped (99.97% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```
```
250184 + 0 in total (QC-passed reads + QC-failed reads)
249964 + 0 primary
0 + 0 secondary
220 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
250108 + 0 mapped (99.97% : N/A)
249888 + 0 primary mapped (99.97% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)

```
</details> 

5. Индексируем и сортируем .bam файл `samtools sort alignment.bam -o alignment_sorted.bam`, `samtools index alignment_sorted.bam`.

## Поиск нуклеотидных различий между референсом и ридами
1. Создаем промежуточный mpileup файл `samtools mpileup -d 361349 -f ../rowData/sequence.fasta alignment_sorted.bam > my.mpileup`.

<details>
<summary>Output:</summary>
 
```
.bam >  my.mpileup
[mpileup] 1 samples in 1 input files
```
</details> 

2. Создаем .vcf файл, используя программу **varscan** `varscan mpileup2snp my.mpileup --min-var-freq 0.001 --variants --output-vcf 1 > varscan_results.vcf`.

<details>
<summary>Output:</summary>
 
```
Only SNPs will be reported
Warning: No p-value threshold provided, so p-values will not be calculated
Min coverage:	8
Min reads2:	2
Min var freq:	0.001
Min avg qual:	15
P-value thresh:	0.01
Reading input from my.mpileup
1665 bases in pileup file
23 variant positions (21 SNP, 2 indel)
0 were failed by the strand-filter
21 variant positions reported (21 SNP, 0 indel)
```
</details> 
