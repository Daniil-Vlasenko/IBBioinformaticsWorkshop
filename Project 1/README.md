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
2. Запускаем программу **fastqc** с новыми данными `fastqc -o . amp_res_baseout_1P amp_res_baseout_2P`. Число ридов сократилось до 445689, больше нет ошибок типа **Per base sequence quality**.
