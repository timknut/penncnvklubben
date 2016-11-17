# penncnvklubben

Links:

* [GenomeStudio 2.0](http://support.illumina.com/array/array_software/genomestudio/downloads.html)
* [PennCNV](http://penncnv.openbioinformatics.org)
* [BeerWare](https://en.wikipedia.org/wiki/Beerware)

## Step -1 Environment setup

```bash
# set up path and directories
export PATH=$PATH:/mnt/users/gjuvslan/PennCNV-1.0.3/   #PennCNV installed here
export penndir=/mnt/users/gjuvslan/penncnvklubben      #git repo cloned here
export datadir=/mnt/users/gjuvslan/penncnvklubben/data #data to analyse collected here
```

## Step 0 GenomeStudio project to input files

* Load .bsf file, Analysis -> Reports -> Report wizard -> Final report
* Displayed fields: SNP name, Sample ID, B Allele freq, Log R ratio. 

![Genome Studio report generation screenshot](https://github.com/argju/penncnvklubben/blob/master/screenshots/report.png)

```bash
#filenames for files from Genome Studio
reportfile="180615_bovine777K_48samples_FinalReport_PennCNV.txt" #Report exported from GenomeStudio
snpposfile="snp_chr_pos.txt"  #Copied columns form Genostudio

#create signal intensity and and pfb files
cd $datadir
mkdir -p signal
split_illumina_report.pl --prefix signal/ --suffix .txt $reportfile   #create signal intensity files
ls -1 signal/*.txt > signalfilelist
compile_pfb.pl --listfile signalfilelist -snpposfile $snpposfile -output pfbfile.pfb #create pdf file
```
