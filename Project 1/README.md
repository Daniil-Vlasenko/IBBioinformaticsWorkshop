# Project 1. “What causes antibiotic resistance?” Alignment to reference, variant calling.
Создаем рабочую деррикторию **Project1**.

## Подготовка данных
1. Создаем дирректорию для сырых данных **rowData** в **Project1** и переходим в нее.
2. Скачиваем референсную последовательность ДНК родительского (неустойчивого к антибиотику) штама `wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.fna.gz` и разархивируем скаченный файл `gunzip GCF_000005845.2_ASM584v2_genomic.fna.gz`.
3. Скачиваем анотацию родительской последовательности `wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.gff.gz` и разархивируем скаченный файл `gunzip GCF_000005845.2_ASM584v2_genomic.gff.gz`.
4. Скачиваем результаты секвенирования нового штама, устойчивого к антибиотику `wget https://figshare.com/ndownloader/files/23769689 -O amp_rez1.fastq.gz` и `wget https://figshare.com/ndownloader/files/23769692  -O amp_rez1.fastq.gz`, разархивируем файлы `gunzip amp_res_1.fastq.gz`, `gunzip amp_res_2.fastq.gz`.
5. Проверим сколько ридов в скачанных файлах `wc -l amp_res_1.fastq`, `wc -l amp_res_2.fastq`. Оба запроса возвращают число 1823504, на каждый рид уходит 4 строки, следовательно, всего 455876 ридов.

## Проверка необработанных данных
1. Запускаем программу **fastqc** с fastq файлами `fastqc -o . amp_res_1.fastq amp_res_2.fastq`. Можно заметить ошибку типа **Per base sequence quality** для обоих файлов, связанную с ухудшением качества хвостов последовательностей, и ошибку типа **Per tile sequence quality** для первого файла. Кажется, первую ошибку можно решить укоротив результаты секвенирования справа, вторую — удалением проблемных последовательностей из данных.

## Исправление данных секвенирования
1. Используем программу **trimmomatic** для исправления данных секвенирования `trimmomatic PE amp_res_1.fastq amp_res_2.fastq -baseout ../processedData/amp_res_baseout SLIDINGWINDOW:10:20 LEADING:20 TRAILING:20 MINLEN:20`.

<details>
<summary>Output:</summary>
 
```
TrimmomaticPE: Started with arguments:
 amp_res_1.fastq amp_res_2.fastq -baseout ../processedData/amp_res_baseout SLIDINGWINDOW:10:20 LEADING:20 TRAILING:20 MINLEN:20
Multiple cores found: Using 4 threads
Using templated Output files: ../processedData/amp_res_baseout_1P ../processedData/amp_res_baseout_1U ../processedData/amp_res_baseout_2P ../processedData/amp_res_baseout_2U
Quality encoding detected as phred33
Input Read Pairs: 455876 Both Surviving: 445689 (97,77%) Forward Only Surviving: 9758 (2,14%) Reverse Only Surviving: 284 (0,06%) Dropped: 145 (0,03%)
TrimmomaticPE: Completed successfully
```
 </details>

2. Переходим в папку **processedData** и запускаем программу **fastqc** с новыми данными `fastqc -o . amp_res_baseout_1P amp_res_baseout_2P`. Число ридов сократилось до 445689, больше нет ошибок типа **Per base sequence quality**.

## Выравнивание ридов на референс
### Индексирование референсной последовательности
1. Переходим в папку **rowData** и индексируем референсную последовательность с помощью программы **bwa** `bwa index GCF_000005845.2_ASM584v2_genomic.fna`.

<details>
<summary>Output:</summary>
 
```
[bwa_index] Pack FASTA... 0.05 sec
[bwa_index] Construct BWT for the packed sequence...
[bwa_index] 1.43 seconds elapse.
[bwa_index] Update BWT... 0.04 sec
[bwa_index] Pack forward-only FASTA... 0.03 sec
[bwa_index] Construct SA from BWT and Occ... 0.30 sec
[main] Version: 0.7.17-r1188
[main] CMD: bwa index GCF_000005845.2_ASM584v2_genomic.fna
[main] Real time: 1.918 sec; CPU: 1.863 sec
```
 </details>
 
### Выравнивание ридов
1. Выполняем выравнивание с помощью bwa `bwa mem GCF_000005845.2_ASM584v2_genomic.fna ../processedData/amp_res_baseout_1P ../processedData/amp_res_baseout_2P > ../processedData/alignment.sam`.

<details>
<summary>Output:</summary>

 ``` 
[M::bwa_idx_load_from_disk] read 0 ALT contigs
[M::process] read 106290 sequences (10000002 bp)...
[M::process] read 108208 sequences (10000042 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (9, 51226, 0, 22)
[M::mem_pestat] skip orientation FF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (145, 185, 230)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 400)
[M::mem_pestat] mean and std.dev: (189.29, 63.57)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 485)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation RR...
[M::mem_pestat] (25, 50, 75) percentile: (114, 158, 263)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 561)
[M::mem_pestat] mean and std.dev: (131.82, 63.27)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 710)
[M::mem_pestat] skip orientation RR
[M::mem_process_seqs] Processed 106290 reads in 2.845 CPU sec, 2.778 real sec
[M::process] read 108830 sequences (10000036 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (20, 51932, 0, 17)
[M::mem_pestat] analyzing insert size distribution for orientation FF...
[M::mem_pestat] (25, 50, 75) percentile: (80, 107, 210)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 470)
[M::mem_pestat] mean and std.dev: (119.00, 69.53)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 600)
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (143, 182, 228)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 398)
[M::mem_pestat] mean and std.dev: (187.09, 63.02)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 483)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation RR...
[M::mem_pestat] (25, 50, 75) percentile: (89, 176, 483)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 1271)
[M::mem_pestat] mean and std.dev: (169.14, 114.54)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 1665)
[M::mem_pestat] skip orientation FF
[M::mem_pestat] skip orientation RR
[M::mem_process_seqs] Processed 108208 reads in 2.969 CPU sec, 2.801 real sec
[M::process] read 106008 sequences (10000061 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (11, 52350, 0, 9)
[M::mem_pestat] analyzing insert size distribution for orientation FF...
[M::mem_pestat] (25, 50, 75) percentile: (100, 135, 178)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 334)
[M::mem_pestat] mean and std.dev: (144.00, 71.73)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 431)
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (142, 181, 226)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 394)
[M::mem_pestat] mean and std.dev: (185.63, 62.35)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 478)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] skip orientation RR as there are not enough pairs
[M::mem_pestat] skip orientation FF
[M::mem_process_seqs] Processed 108830 reads in 3.020 CPU sec, 2.855 real sec
[M::process] read 105838 sequences (10000065 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (13, 51061, 0, 13)
[M::mem_pestat] analyzing insert size distribution for orientation FF...
[M::mem_pestat] (25, 50, 75) percentile: (84, 104, 199)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 429)
[M::mem_pestat] mean and std.dev: (151.77, 93.70)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 544)
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (144, 184, 231)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 405)
[M::mem_pestat] mean and std.dev: (189.09, 64.12)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 492)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation RR...
[M::mem_pestat] (25, 50, 75) percentile: (110, 130, 1006)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 2798)
[M::mem_pestat] mean and std.dev: (475.38, 646.65)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 3694)
[M::mem_pestat] skip orientation FF
[M::mem_pestat] skip orientation RR
[M::mem_process_seqs] Processed 106008 reads in 2.918 CPU sec, 2.755 real sec
[M::process] read 106412 sequences (10000181 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (12, 50847, 0, 11)
[M::mem_pestat] analyzing insert size distribution for orientation FF...
[M::mem_pestat] (25, 50, 75) percentile: (54, 73, 151)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 345)
[M::mem_pestat] mean and std.dev: (90.75, 49.78)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 442)
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (143, 182, 227)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 395)
[M::mem_pestat] mean and std.dev: (186.62, 62.45)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 479)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation RR...
[M::mem_pestat] (25, 50, 75) percentile: (83, 125, 196)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 422)
[M::mem_pestat] mean and std.dev: (118.30, 58.38)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 535)
[M::mem_pestat] skip orientation FF
[M::mem_pestat] skip orientation RR
[M::mem_process_seqs] Processed 105838 reads in 2.953 CPU sec, 2.783 real sec
[M::process] read 107246 sequences (10000180 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (18, 51171, 0, 16)
[M::mem_pestat] analyzing insert size distribution for orientation FF...
[M::mem_pestat] (25, 50, 75) percentile: (89, 126, 175)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 347)
[M::mem_pestat] mean and std.dev: (122.25, 49.00)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 433)
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (144, 183, 229)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 399)
[M::mem_pestat] mean and std.dev: (188.23, 63.47)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 484)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation RR...
[M::mem_pestat] (25, 50, 75) percentile: (52, 105, 220)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 556)
[M::mem_pestat] mean and std.dev: (120.33, 80.63)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 724)
[M::mem_pestat] skip orientation FF
[M::mem_pestat] skip orientation RR
[M::mem_process_seqs] Processed 106412 reads in 3.078 CPU sec, 2.903 real sec
[M::process] read 104380 sequences (10000182 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (11, 51247, 0, 12)
[M::mem_pestat] analyzing insert size distribution for orientation FF...
[M::mem_pestat] (25, 50, 75) percentile: (54, 90, 172)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 408)
[M::mem_pestat] mean and std.dev: (108.73, 60.01)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 526)
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (143, 182, 228)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 398)
[M::mem_pestat] mean and std.dev: (187.06, 62.87)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 483)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation RR...
[M::mem_pestat] (25, 50, 75) percentile: (58, 108, 264)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 676)
[M::mem_pestat] mean and std.dev: (132.64, 94.62)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 882)
[M::mem_pestat] skip orientation FF
[M::mem_pestat] skip orientation RR
[M::mem_process_seqs] Processed 107246 reads in 3.380 CPU sec, 3.210 real sec
[M::process] read 38166 sequences (3590317 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (12, 50331, 0, 10)
[M::mem_pestat] analyzing insert size distribution for orientation FF...
[M::mem_pestat] (25, 50, 75) percentile: (68, 135, 213)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 503)
[M::mem_pestat] mean and std.dev: (120.09, 68.06)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 648)
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (143, 183, 229)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 401)
[M::mem_pestat] mean and std.dev: (188.26, 63.54)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 487)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation RR...
[M::mem_pestat] (25, 50, 75) percentile: (102, 140, 152)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (2, 252)
[M::mem_pestat] mean and std.dev: (116.11, 42.39)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 302)
[M::mem_pestat] skip orientation FF
[M::mem_pestat] skip orientation RR
[M::mem_process_seqs] Processed 104380 reads in 2.978 CPU sec, 2.856 real sec
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (2, 18244, 0, 4)
[M::mem_pestat] skip orientation FF as there are not enough pairs
[M::mem_pestat] analyzing insert size distribution for orientation FR...
[M::mem_pestat] (25, 50, 75) percentile: (141, 179, 224)
[M::mem_pestat] low and high boundaries for computing mean and std.dev: (1, 390)
[M::mem_pestat] mean and std.dev: (184.03, 61.45)
[M::mem_pestat] low and high boundaries for proper pairs: (1, 473)
[M::mem_pestat] skip orientation RF as there are not enough pairs
[M::mem_pestat] skip orientation RR as there are not enough pairs
[M::mem_process_seqs] Processed 38166 reads in 1.163 CPU sec, 1.046 real sec
[main] Version: 0.7.17-r1188
[main] CMD: bwa mem GCF_000005845.2_ASM584v2_genomic.fna ../processedData/amp_res_baseout_1P ../processedData/amp_res_baseout_2P
[main] Real time: 24.261 sec; CPU: 25.459 sec
```
</details>

### Сконвертируем .sam файл в .bam файл
1. Для конвертации перейдем в пвпку **processedData** и используем программу samtools `samtools view -S -b alignment.sam > alignment.bam`.
2. Проверим, сколько ридрв было выравнено `samtools flagstat alignment.bam`.

<details>
<summary>Output:</summary>
 
```
891635 + 0 in total (QC-passed reads + QC-failed reads)
891378 + 0 primary
0 + 0 secondary
257 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
890569 + 0 mapped (99.88% : N/A)
890312 + 0 primary mapped (99.88% : N/A)
891378 + 0 paired in sequencing
445689 + 0 read1
445689 + 0 read2
887530 + 0 properly paired (99.57% : N/A)
889384 + 0 with itself and mate mapped
928 + 0 singletons (0.10% : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```
<details> 

 3. Индексируем и сортируем .bam файл `samtools sort alignment.bam -o alignment_sorted.bam`, `samtools index alignment_sorted.bam`.

