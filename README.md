# penncnvklubben

Links:

* [GenomeStudio 2.0](http://support.illumina.com/array/array_software/genomestudio/downloads.html)
* [PennCNV](http://penncnv.openbioinformatics.org)
* [BeerWare](https://en.wikipedia.org/wiki/Beerware)

## Step 0 GenomeStudio project to intensity files

* Load .bsf file, Analysis -> Reports -> Report wizard -> Final report
* Displayed fields: SNP name, Sample ID, B Allele freq, Log R ratio. 

![Genome Studio report generation screenshot](https://github.com/argju/penncnvklubben/blob/master/screenshots/report.png)

```bash
reportfile="/mnt/c/Users/arneg.GENO/Desktop/GENO\ oppstart/180615_bovine777K_48samples_FinalReport_PennCNV.txt"
mkdir signal
split_illumina_report.pl --prefix signal/ --suffix .txt $reportfile
```

