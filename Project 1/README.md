# Project 1. “What causes antibiotic resistance?” Alignment to reference, variant calling.
Создаем рабочую деррикторию **Project1**.

## Подготовка данных
1. Создаем дирректорию для сырых данных **rowData** в **Project1** и переходим в нее.
2. Скачиваем референсную последовательность ДНК родительского (неустойчивого к антибиотику) штама `wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.fna.gz` и разархивируем скаченный файл `gunzip GCF_000005845.2_ASM584v2_genomic.fna.gz`.
3. Скачиваем анотацию родительской последовательности `wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.gff.gz` и разархивируем скаченный файл `gunzip GCF_000005845.2_ASM584v2_genomic.gff.gz`.
4. Скачиваем результаты секвенирования нового штама, устойчивого к антибиотику `wget https://figshare.com/ndownloader/files/23769689 -O amp_rez1.fastq.gz` и `wget https://figshare.com/ndownloader/files/23769692  -O amp_rez1.fastq.gz`, разархивируем файлы `gunzip amp_res_1.fastq.gz`, `gunzip amp_res_2.fastq.gz`.
5. Проверим сколько ридов в скачанных файлах `wc -l amp_res_1.fastq`, `wc -l amp_res_2.fastq`. Оба запроса возвращают число 1823504, на каждый рид уходит 4 строки, следовательно, всего 455876 ридов.

## Проверка необработанных данных
1. Запустим программу fastqc с fastq файлами `fastqc -o . [path to file 1]/amp_res_1.fastq [path to file 2]/amp_res_2.fastq`.
