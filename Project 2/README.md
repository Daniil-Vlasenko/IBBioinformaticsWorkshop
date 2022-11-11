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

5. Индексируем и сортируем .bam файлы `samtools sort alignment_SRR1705851.bam -o alignment_SRR1705851_sorted.bam`, `samtools sort alignment_SRR1705858.bam -o alignment_SRR1705858_sorted.bam`, `samtools sort alignment_SRR1705859.bam -o alignment_SRR1705859_sorted.bam`, `samtools sort alignment_SRR1705860.bam -o alignment_SRR1705860_sorted.bam` и `samtools index alignment_SRR1705851_sorted.bam`, `samtools index alignment_SRR1705858_sorted.bam`, `samtools index alignment_SRR1705859_sorted.bam`, `samtools index alignment_SRR1705860_sorted.bam`.

## Поиск нуклеотидных различий между референсом и ридами
1. Создаем промежуточные mpileup файлы `samtools mpileup -d 400000 -f ../rowData/sequence.fasta alignment_SRR1705851_sorted.bam > SRR1705851.mpileup`, `samtools mpileup -d 400000 -f ../rowData/sequence.fasta alignment_SRR1705858_sorted.bam > SRR1705858.mpileup`, `samtools mpileup -d 400000 -f ../rowData/sequence.fasta alignment_SRR1705859_sorted.bam > SRR1705859.mpileup`, `samtools mpileup -d 400000 -f ../rowData/sequence.fasta alignment_SRR1705860_sorted.bam > SRR1705860.mpileup`.
2. Создаем .vcf файлы, используя программу **varscan** `varscan mpileup2snp SRR1705851.mpileup --min-var-freq 0.001 --variants --output-vcf 1 > SRR1705851_varscan_results.vcf`, `varscan mpileup2snp SRR1705858.mpileup --min-var-freq 0.001 --variants --output-vcf 1 > SRR1705858_varscan_results.vcf`, `varscan mpileup2snp SRR1705859.mpileup --min-var-freq 0.001 --variants --output-vcf 1 > SRR1705859_varscan_results.vcf`, `varscan mpileup2snp SRR1705860.mpileup --min-var-freq 0.001 --variants --output-vcf 1 > SRR1705860_varscan_results.vcf`.

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
Reading input from SRR1705851.mpileup
1665 bases in pileup file
23 variant positions (21 SNP, 2 indel)
0 were failed by the strand-filter
21 variant positions reported (21 SNP, 0 indel)
```
```
Only SNPs will be reported
Warning: No p-value threshold provided, so p-values will not be calculated
Min coverage:	8
Min reads2:	2
Min var freq:	0.001
Min avg qual:	15
P-value thresh:	0.01
Reading input from SRR1705858.mpileup
1665 bases in pileup file
58 variant positions (58 SNP, 0 indel)
1 were failed by the strand-filter
57 variant positions reported (57 SNP, 0 indel)
```
```
Only SNPs will be reported
Warning: No p-value threshold provided, so p-values will not be calculated
Min coverage:	8
Min reads2:	2
Min var freq:	0.001
Min avg qual:	15
P-value thresh:	0.01
Reading input from SRR1705859.mpileup
1665 bases in pileup file
54 variant positions (54 SNP, 0 indel)
2 were failed by the strand-filter
52 variant positions reported (52 SNP, 0 indel)
```
```
Only SNPs will be reported
Warning: No p-value threshold provided, so p-values will not be calculated
Min coverage:	8
Min reads2:	2
Min var freq:	0.001
Min avg qual:	15
P-value thresh:	0.01
Reading input from SRR1705860.mpileup
1665 bases in pileup file
61 variant positions (61 SNP, 0 indel)
0 were failed by the strand-filter
61 variant positions reported (61 SNP, 0 indel)
```
</details> 

3. Извлекаем необходимую для анализа информацию из контрольных данных `awk 'BEGIN{FS="\t|:"; OFS=" "} {if(NR>24) {gsub("%","",$29); print $2,$4,$5,$29}}' SRR1705858_varscan_results.vcf > SRR1705858_varscan_results_awk.txt`, `awk 'BEGIN{FS="\t|:"; OFS=" "} {if(NR>24) {gsub("%","",$29); print $2,$4,$5,$29}}' SRR1705859_varscan_results.vcf > SRR1705859_varscan_results_awk.txt`, `awk 'BEGIN{FS="\t|:"; OFS=" "} {if(NR>24) {gsub("%","",$29); print $2,$4,$5,$29}}' SRR1705860_varscan_results.vcf > SRR1705860_varscan_results_awk.txt`.

В итоге имеем SNPs вируса больного, среди которых будем искать настоящие мутации, и SNPs, полученные через секвенирование референсного вируса, которые являются шумами. Будем обращать внимание на SNPs больного, если частота соответствующего SNP отклоняется от средней частоты шума больше чем на 3 стандартных отклонения шума (Правило трех сигм). Ниже представлены SNPs больного:
```
72 A G 99.96
117 C T 99.82
254 A G 0.17
276 A G 0.17
307 C T 0.94
340 T C 0.17
389 T C 0.22
691 A G 0.17
722 A G 0.2
744 A G 0.17
774 T C 99.96
802 A G 0.23
859 A G 0.18
915 T C 0.19
999 C T 99.86
1043 A G 0.18
1086 A G 0.21
1213 A G 0.22
1260 A C 99.94
1280 T C 0.18
1458 T C 0.84
```
Средние частот шумов: `0.2564912`, `0.2369231`, `0.2503279`. Стандартные отклонения частот шумов: `0.07172595`, `0.05237641`, `0.07803775`. Соответственно среднее средних частот шумов: `0.2479141`. Среднее стандартных отклонений частот шумов: `0.06738004`. Последние два числа будем использовать в качестве среднего и стандартного отклонения шума.

Остаются слудующие SNPs:
```
72 A G 99.96
117 C T 99.82
307 C T 0.94
774 T C 99.96
999 C T 99.86
1260 A C 99.94
1458 T C 0.84
```

Из них не молчащей мутацией является только  `307 C T 0.94 Pro->Ser`.

## Картирование эпитопов
Судя по статье https://drive.google.com/file/d/1xe5-4LxIV4bO4mX6jhvrMAqtpOpkWsXm/view, найденная мутация влияет на участок 103 эпитопа D белка HA вируса, что и могло привести к неспособности антибиотика связваться с новым штамом вируса.
