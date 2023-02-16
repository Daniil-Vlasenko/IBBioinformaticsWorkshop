# Project 5. H+, or how to build a perfect human.
Создаем рабочую директорию **Project5** и в ней папку для сырых **rowData** и обработанных данных **processedData**

## Подготовка данных
1. Скачиваем данные сырые данные с https://drive.google.com/open?id=1QJkwJe5Xl_jSVpqdTSNXP7sqlYfI666j.
2. C помощью **plink** конвертируем сырые данные в vcf формат `plink --23file <23_and_me_input.txt> --recode vcf --out snps_clean --output-chr MT --snps-only just-acgt`.

## Определение родительских гаплогрупп.
1. С помощью **James Lick's mtHap utility** анализируем мтДНК и определяем материнскую гаплогруппу. [SNP_raw_v4_Full_20170514175358—mtDNA Haplogroup Analysis Report.pdf](https://github.com/Daniil-Vlasenko/IBBioinformaticsWorkshop/files/10757570/SNP_raw_v4_Full_20170514175358.mtDNA.Haplogroup.Analysis.Report.pdf)
2. С помощью **MorleyDNA.com Y-SNP Subclade Predictor** анализирум Y-хромосому и определяем отцовскую гаплогруппу. [MorleyDNA.com Y-SNP Subclade Predictor.pdf](https://github.com/Daniil-Vlasenko/IBBioinformaticsWorkshop/files/10757583/MorleyDNA.com.Y-SNP.Subclade.Predictor.pdf)

## Определение пола и цвета глаз.
1. Пол, очевидно, мужской, так как в данных есть Y-хромосома.
2. Цвет глаз. В соответствии с этой статьей https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3694299/ будет не голубой так как rs12913832 = AG, по остальным SNP's нет точной информации, поэтому предполагаем, что цвет глаз будет между зеленым и коричневым.

## Аннотирование всех SNP, выбор клинически значимых.
1. Для анотирования используем Variant Effect Predictor.
2. 
