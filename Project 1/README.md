# Project 1. “What causes antibiotic resistance?” Alignment to reference, variant calling.
Создаем рабочую деррикторию **Project1**.

## Подготовка данных
1. Создаем дирректорию для сырых данных **rowData** в **Project1**.
2. Скачиваем референсную последовательность ДНК родительского (неустойчивого к антибиотику) штама `wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.fna.gz` и разархивируем скаченный файл `gunzip GCF_000005845.2_ASM584v2_genomic.fna.gz`.
3. Скачиваем анотацию родительской последовательности `wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.gff.gz` и разархивируем скаченный файл `gunzip GCF_000005845.2_ASM584v2_genomic.gff.gz`.
4. Скачиваем результаты секвенирования нового штама, устойчивого к антибиотику `wget https://doi.org/10.6084/m9.figshare.10006541.v3`.
