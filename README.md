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
reportfile1="180615_bovine777K_48samples_FinalReport_PennCNV.txt" #Report exported from GenomeStudio 48 samples
reportfile2="150430_Tim_777K_144samples_recluster_FinalReport.txt" #Report exported from GenomeStudio 144 samples
snpposfile="snp_chr_pos.txt"  #Copied columns form Genostudio

#create signal intensity and and pfb files
cd $datadir
mkdir -p signal
split_illumina_report.pl --prefix signal/ --suffix .txt $reportfile1   #create signal intensity files
split_illumina_report.pl --prefix signal/ --suffix .txt $reportfile2   #create signal intensity files
ls -1 signal/*.txt > signalfilelist
compile_pfb.pl --listfile signalfilelist -snpposfile $snpposfile -output pfbfile.pfb #create pdf file
```

## Step 1 Detect CNVs

```bash
#cnv detection
detect_cnv.pl --test -pfb pfbfile.pfb -hmm ../../PennCNV-1.0.3/lib/hhall.hmm --lastchr 29 signal/9200246_7736.txt signal/6333982_8239.txt --log detect_cnv.log --out detect_cnv.out #two signal files
detect_cnv.pl --test -pfb pfbfile.pfb -hmm ../../PennCNV-1.0.3/lib/hhall.hmm --lastchr 29 --listfile signalfilelist --log detect_cnv.log --out detect_cnv.out #all signal files

#quality control
filter_cnv.pl detect_cnv.out -qclogfile detect_cnv.log -qclrrsd 0.2 -qcnumcnv 50 -qcpassout detect_cnv.qcpass -qcsumout detect_cnv.qcsum -out detect_cnv.goodcnv
```

## Step :v: Explore results

```R
library(data.table)
library(stringr)
library(plotly)

#Read into R
cnv <- fread('sed -E s/[[:space:]]+/" "/g ~/penncnvklubben/data/detect_cnv.out',header=F,sep=" ")
cnv[,numsnp:=as.numeric(str_replace(V2,'numsnp=',''))]
cnv[,length:=as.numeric(str_replace_all(str_replace(V3,'length=',''),',',''))]
cnv[,id:=str_replace(str_replace(V5,'signal/',''),'.txt','')]
cnv[,startsnp:=str_replace(V6,'startsnp=','')]
cnv[,endsnp:=str_replace(V7,'endsnp=','')]
pos <- str_split(str_replace(cnv$V1,'chr',''),'[:,-]',simplify=T)
cnv$chr <- as.numeric(pos[,1])
cnv$start <- as.numeric(pos[,2])
cnv$end <- as.numeric(pos[,3])
statecn <- str_split(str_replace(str_replace(cnv$V4,'state',''),'cn=',''),'[,]',simplify=T)
cnv$state <- statecn[,1]
cnv$cn <- statecn[,2]
cnv <- cnv[,.(chr,start,end,length,numsnp,id,state,cn,startsnp,endsnp)]


#Plot results
ggplot(cnv[chr==3]) + geom_linerange(aes(x=id,ymin=start,ymax=end)) + coord_flip() 
ggplot(cnv[chr==5]) + geom_linerange(aes(x=id,ymin=start,ymax=end)) + coord_flip() 
```
