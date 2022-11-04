# Project 2. “Why did I get the flu?”. Deep sequencing, error control, p-value, viral evolution.
Создаем рабочую директорию **Project2**.

## Подготовка данных
1. Создаем директорию для сырых данных **rowData** в **Project2** и переходим в нее.
2. Скачиваем данные секвенирования, полученные от болльного `wget http://ftp.sra.ebi.ac.uk/vol1/fastq/SRR170/001/SRR1705851/SRR1705851.fastq.gz`. и разархивируем файл `gunzip SRR1705851.fastq.gz`.
3. Скачиваем референсную последовательность гена вируса с https://www.ncbi.nlm.nih.gov/nuccore/KF848938.1?report=fasta.
4. Проверим сколько ридов в скачанном файлах `wc -l SRR1705851.fastq`. Запрос возвращает число 1433060, на каждый рид уходит 4 строки, следовательно, всего 358265 ридов.

## Проверка необработанных данных
1. Запускаем программу **fastqc** с fastq файлами `fastqc -o . SRR1705851.fastq`. Можно заметить ошибку типа **Per base sequence content** и ошибку типа **Sequence Duplication Levels**, оставим данные такими.

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
 
2. Выполняем выравнивание ридов с помощью bwa `bwa mem sequence.fasta SRR1705851.fastq > ../processedData/alignment.sam`.

<details>
<summary>Output:</summary>

 ``` 
1705851.fastq > ../processedData/alignment.sam
[M::bwa_idx_load_from_disk] read 0 ALT contigs
[M::process] read 68388 sequences (10000129 bp)...
[M::process] read 67628 sequences (10000233 bp)...
[M::mem_process_seqs] Processed 68388 reads in 1.503 CPU sec, 1.465 real sec
[M::process] read 67698 sequences (10000046 bp)...
[M::mem_process_seqs] Processed 67628 reads in 1.237 CPU sec, 1.172 real sec
[M::process] read 67652 sequences (10000169 bp)...
[M::mem_process_seqs] Processed 67698 reads in 1.572 CPU sec, 1.508 real sec
[M::process] read 68072 sequences (10000295 bp)...
[M::mem_process_seqs] Processed 67652 reads in 1.516 CPU sec, 1.456 real sec
[M::process] read 18827 sequences (2716992 bp)...
[M::mem_process_seqs] Processed 68072 reads in 1.302 CPU sec, 1.260 real sec
[M::mem_process_seqs] Processed 18827 reads in 0.570 CPU sec, 0.536 real sec
[main] Version: 0.7.17-r1188
[main] CMD: bwa mem sequence.fasta SRR1705851.fastq
[main] Real time: 7.534 sec; CPU: 7.768 sec
```
</details>

3. Конвертируем .sam файл в .bam файл. Для этого перейдем в директорию **processedData** и используем программу **samtools** `samtools view -S -b alignment.sam > alignment.bam`.
4. Проверим, сколько ридрв было выравнено `samtools flagstat alignment.bam`.

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
</details> 

5. Индексируем и сортируем .bam файл `samtools sort alignment.bam -o alignment_sorted.bam`, `samtools index alignment_sorted.bam`.

## Поиск нуклеотидных различий между референсом и ридами
1. Создаем промежуточный mpileup файл `samtools mpileup -f ../rowData/sequence.fasta alignment_sorted.bam >  my.mpileup`.

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
12 variant positions (10 SNP, 2 indel)
1 were failed by the strand-filter
9 variant positions reported (9 SNP, 0 indel)
```
</details> 
