# Project 3. "E. coli outbreak investigation". De novo assembly and annotation of bacterial genomes.
Создаем рабочую директорию **Project3**.

## Подготовка данных
1. Создаем директорию для сырых данных **rowData** в **Project1** и переходим в нее.
2. Скачиваем прямые и обратные риды  длины 470 bp, 2 kb and 6 kb `wget https://d28rh4a8wq0iu5.cloudfront.net/bioinfo/SRR292678sub_S1_L001_R1_001.fastq.gz https://d28rh4a8wq0iu5.cloudfront.net/bioinfo/SRR292678sub_S1_L001_R2_001.fastq.gz https://d28rh4a8wq0iu5.cloudfront.net/bioinfo/SRR292862_S2_L001_R1_001.fastq.gz https://d28rh4a8wq0iu5.cloudfront.net/bioinfo/SRR292862_S2_L001_R2_001.fastq.gz https://d28rh4a8wq0iu5.cloudfront.net/bioinfo/SRR292770_S1_L001_R1_001.fastq.gz https://d28rh4a8wq0iu5.cloudfront.net/bioinfo/SRR292770_S1_L001_R2_001.fastq.gz` и разархивирываем их `gunzip RR292678sub_S1_L001_R1_001.fastq.gz SRR292770_S1_L001_R2_001.fastq.gz SRR292678sub_S1_L001_R2_001.fastq.gz SRR292862_S2_L001_R1_001.fastq.gz SRR292770_S1_L001_R1_001.fastq.gz SRR292862_S2_L001_R2_001.fastq.gz`.
3. Проверим сколько ридов в файлах. Запускаем программу **fastqc** с fastq файлами `fastqc -o . RR292678sub_S1_L001_R1_001.fastq.gz SRR292770_S1_L001_R2_001.fastq.gz SRR292678sub_S1_L001_R2_001.fastq.gz SRR292862_S2_L001_R1_001.fastq.gz SRR292770_S1_L001_R1_001.fastq.gz SRR292862_S2_L001_R2_001.fastq.gz`.
