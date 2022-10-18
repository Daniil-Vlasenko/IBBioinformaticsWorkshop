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
1. Используем программу **trimmomatic** для исправления данных секвенирования `trimmomatic PE amp_res_1.fastq amp_res_2.fastq -baseout ../processedData/amp_res_baseout SLIDINGWINDOW:10:20 LEADING:20 TRAILING:20 MINLEN:20`:
```
TrimmomaticPE: Started with arguments:
 amp_res_1.fastq amp_res_2.fastq -baseout ../processedData/amp_res_baseout SLIDINGWINDOW:10:20 LEADING:20 TRAILING:20 MINLEN:20
Multiple cores found: Using 4 threads
Using templated Output files: ../processedData/amp_res_baseout_1P ../processedData/amp_res_baseout_1U ../processedData/amp_res_baseout_2P ../processedData/amp_res_baseout_2U
Quality encoding detected as phred33
Input Read Pairs: 455876 Both Surviving: 445689 (97,77%) Forward Only Surviving: 9758 (2,14%) Reverse Only Surviving: 284 (0,06%) Dropped: 145 (0,03%)
TrimmomaticPE: Completed successfully
```
2. Переходим в папку **processedData** и запускаем программу **fastqc** с новыми данными `fastqc -o . amp_res_baseout_1P amp_res_baseout_2P`. Число ридов сократилось до 445689, больше нет ошибок типа **Per base sequence quality**.

## Выравнивание ридов на референс
### Индексирование референсной последовательности
1. Переходим в папку **rowData** и индексируем референсную последовательность с помощью программы **bwa** `bwa index GCF_000005845.2_ASM584v2_genomic.fna`.
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
### Выравнивание ридов
1. Выполняем выравнивание с помощью bwa `bwa mem GCF_000005845.2_ASM584v2_genomic.fna ../processedData/amp_res_baseout_1P ../processedData/amp_res_baseout_2P > ../processedData/alignment.sam`.
```
[M::bwa_idx_load_from_disk] read 0 ALT contigs
[M::process] read 106290 sequences (10000002 bp)...
[M::process] read 108208 sequences (10000042 bp)...
[M::mem_pestat] # candidate unique pairs for (FF, FR, RF, RR): (9, 51226, 0, 22)
[M::mem_pestat] skip orientation FF as there are not enough pairs
...
[M::mem_pestat] skip orientation RR as there are not enough pairs
[M::mem_process_seqs] Processed 38166 reads in 1.163 CPU sec, 1.046 real sec
[main] Version: 0.7.17-r1188
[main] CMD: bwa mem GCF_000005845.2_ASM584v2_genomic.fna ../processedData/amp_res_baseout_1P ../processedData/amp_res_baseout_2P
[main] Real time: 24.261 sec; CPU: 25.459 sec
```





