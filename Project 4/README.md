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
4. Извлечем из файла, содержащего все белки, белки, которые были найдены с помощью выравнивания пептидов:
```
allProt = open("../processedData/augustus.whole.fna", "r")
interestProt = open("../processedData/blast_output.txt", "r")
outputFile = open("../processedData/proteins_after_localization.fna", "w")

interestProtList = []
for l in interestProt:
    interestProtList.append(l.split()[1])

isInterestProtein = False
count = 0
for l in allProt:
    if l[0] == ">":
        if l[1:-1] in interestProtList:
            isInterestProtein = True
            count += 1
        else:
            isInterestProtein = False
    if isInterestProtein:
        outputFile.write(l)
```

## Предположение локализации белков
1. Используем **WoLF PSORT** для локализации белков, полученных на прошлом шаге — [WoLF PSORT Prediction.pdf](https://github.com/Daniil-Vlasenko/IBBioinformaticsWorkshop/files/10236580/WoLF.PSORT.Prediction.pdf).
2. Выбираем все белки, которые потенциально могут быть ядерными — [proteins.txt](https://github.com/Daniil-Vlasenko/IBBioinformaticsWorkshop/files/10237163/proteins.txt).

## Blast поиск
1. Выполняем Blast поиск по всем белкам, которые были найдены на прошлом шаге. В следующей таблице представлены те белки, которые нашел Blast.

| Accession Number | E-value | % Ident | % Query coverage | Annotation |
| :---: | :---: | :---: | :---: | :---: |
| g2203.t1 | 2e-126 | 35.93% | 75% | Q69ZQ1.2 |
| g3428.t1 | 9e-65	| 56.60% | 91% | Q09510.1 |
| g5927.t1 | 1e-18 | 38.64% | 14% | Q17427.1 |
| g7861.t1 | 2e-71 | 37.21% | 99% | B4F769.1 |
| g8100.t1 | 3e-46 | 36.04% | 22% | Q2YDR3.1 |
| g8312.t1 | 0.0 | 40.84% | 84% | Q5KU39.1 |
| g11513.t1 | 7e-83 | 28.61% | 68% | Q32PH0.1 |
| g11960.t1 | 6e-98 | 26.96% | 96% | Q8CJB9.1 |
| g14472.t1 | 0.0 | 100.00% | 100% | P0DOW4.1 |
| g15484.t1 | 0.0 | 45.03% | 78% | Q155U0.1 |
| g16318.t1 | 4e-08 | 36.11% | 40% | A2VD00.1 |
| g16368.t1 | 1e-05 | 39.29% | 35% | A4II09.1 |

## HHMMER поиск
1. Выполняем HHMMER поиск по всем белкам, которые были найдены на предыдущем шаге - [score results _ HMMER.pdf](https://github.com/Daniil-Vlasenko/IBBioinformaticsWorkshop/files/10238527/score.results._.HMMER.pdf). В следующей таблице представлены белки, которые нашел HAMMER.

| Accession Number | Annotation | Conditional E-value | Independent E-value |
| :---: | :---: | :---: | :---: |
| g2203.t1 | Glycosyl hydrolases family 31 | 2.4e-45 | 4.8e-41 |
| g7861.t1 | SNF2-related domain | 1.8e-32 | 1.2e-28 |
| g8100.t1 | Inositol monophosphatase family | 2.0e-41 | 1.9e-37 |
| g8312.t1 | Region in Clathrin and VPS | 2.7e-27 | 5.4e-23 |
| g11513.t1 | Transport protein Trs120 or TRAPPC9, TRAPP II complex subunit | 4.9e-14 | 9.6e-10 |
| g11960.t1 | Zinc finger, C3HC4 type (RING finger) | 2.1e-09 | 4.2e-05 |
| g15484.t1 | Vps51/Vps67 | 3.2e-27 | 1.3e-23 |

