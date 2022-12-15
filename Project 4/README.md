# Project 4. Tardigrades: from genestealers to space marines
Создаем рабочую директорию **Project4**.

## Подготовка данных
1. Создаем директории для сырых и обработанных данных **rowData** и **processedData** в **Project4**.
2. Скачиваем уже собранный геном тихоходки `wget http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/949/185/GCA_001949185.1_Rvar_4.0/GCA_001949185.1_Rvar_4.0_genomic.fna.gz` и разархивирываем его `gunzip GCA_001949185.1_Rvar_4.0_genomic.fna.gz`.
3. Скачиваем фунциональную анотацию генома, выполненную с помощью **Augustus** https://drive.google.com/file/d/1wBxf6cDgu22NbjAOgTe-8b3Zx60hNKY0/view?usp=drive_web.
4. Вытаскиваем анотации генома последовательности белков в формате фаста с помощью https://github.com/nextgenusfs/augustus/blob/master/scripts/getAnnoFasta.pl `perl getAnnoFasta.pl processedData/augustus.whole.gff`.
5. Посчитаем, сколько белков мы получили `grep -o '>' augustus.whole.fna | wc -l`. Итог — 16435.

## Физическая локализация
1. Скачиваем результаты масс-спектрометрии хроматина тихоходки https://disk.yandex.ru/d/xJqQMGX77Xueqg.
2. Создаем базу данных пептидов с помощью **blast** `makeblastdb -in augustus.whole.fna -dbtype prot  -out blast_database`.
3. Выравниванием данный масс-спектра на базу данных `blastp -db blast_database -query ../rowData/peptides.fa -outfmt 6  -out blast_output`.

## Предположение локализации белков

1. Используем **WoLF PSORT** для локализации белков, полученных на прошлом шаге — [WoLF PSORT Prediction.pdf](https://github.com/Daniil-Vlasenko/IBBioinformaticsWorkshop/files/10236580/WoLF.PSORT.Prediction.pdf)
