# bior_annotate tutorial

### Prerequisties

#### Download Reference files (this may take some time depending on your connection speed)

```
#Get human reference genome
mkdir references
curl -# -o references/hg19.fa.gz ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/phase2_reference_assembly_sequence/hs37d5.fa.gz 
bgzip -dc references/hg19.fa.gz > references/hg19.fa
samtools faidx references/hg19.fa
```

#### Download the `test.vcf` file.

```
wget https://raw.githubusercontent.com/Steven-N-Hart/bior_annotate/master/trunk/test.vcf
```

#### Get bior catalogs 

```
mkdir catalogs && cd catalogs
# Download the 1000 genomes catalog
curl -# -O https://s3-us-west-2.amazonaws.com/mayo-bic-tools/bior_annotate/catalogs/chr17/1000_genomes_chr17.tar.gz
# Download the ClinVar catalog
curl -# -O https://s3-us-west-2.amazonaws.com/mayo-bic-tools/bior_annotate/catalogs/chr17/ClinVar_chr17.tar.gz
# Download the Exome Sequencing Project catalog
curl -# -O https://s3-us-west-2.amazonaws.com/mayo-bic-tools/bior_annotate/catalogs/chr17/ESP_chr17.tar.gz
# Download the dbSNP catalog
curl -# -O https://s3-us-west-2.amazonaws.com/mayo-bic-tools/bior_annotate/catalogs/chr17/dbSNP_chr17.tar.gz

# Now unzip the catalogs
for x in *tar.gz
do
  tar xvzf $x
  rm $x
done

#Change back to the home directory
cd ..
```

Your `catalogs` directory should now look like this:

```
1000_genomes/
    20130502_GRCh37/
        variants_nodups.v1/
            ALL.wgs.sites.vcf.columns.tsv
            ALL.wgs.sites.vcf.datasource.properties
            ALL.wgs.sites.vcf.tsv.bgz
            ALL.wgs.sites.vcf.tsv.bgz.tbi

ClinVar/
    20160515_GRCh37/
        variants_nodups.v1/
            macarthur-lab_xml_txt.columns.tsv
            macarthur-lab_xml_txt.columns.tsv.blacklist #Note: this file is not used here
            macarthur-lab_xml_txt.columns.tsv.blacklist.biorweb #Note: this file is not used here
            macarthur-lab_xml_txt.datasource.properties
            macarthur-lab_xml_txt.tsv.bgz
            macarthur-lab_xml_txt.tsv.bgz.tbi

dbSNP/
    139/
        chr17_GRCh37.columns.tsv
        chr17_GRCh37.datasource.properties
        chr17_GRCh37.tsv.bgz
        chr17_GRCh37.tsv.bgz.tbi
    142_GRCh37.p13/
        variants_nodups.v1
            chr17.vcf.columns.tsv
            chr17.vcf.datasource.properties
            chr17.vcf.tsv.bgz
            chr17.vcf.tsv.bgz.tbi
ESP/
    V2_GRCh37/
        variants.nodups.v1/
            ESP6500.vcf.columns.tsv
            ESP6500.vcf.datasource.properties
            ESP6500.vcf.tsv.bgz
            ESP6500.vcf.tsv.bgz.tbi
```
> Notice with this directory structure, you can keep multiple versions of the same source availalble (e.g. dbSNP).  You don't even need to keep them all in the same directory structure as shown.


#### Configuration
`bior_annotate.sh` requires 2 configuration files: a `catalog.file` and a `drill.file`.

The catalog file tells bior_annotate.sh where the annotation catalogs are located on your filesystem. 

 * column 1 is the ShortUniqueName in the *datasource.properties file for that catalog
 * column 2 is what bior command you wish to run [e.g. bior_overlap or bior_same_variant]
 * column 3 is the path to the catalog

Create your own catalog file for the annotation sets you just downloaded by copying & pasting this information:

```
ExAC_r03_GRCh37_nodups  bior_same_variant   /Data/catalogs/2015_05_18/ExAC.r0.3.sites.vep.vcf.tsv.bgz
1000genomes_20130502_GRCh37_nodups  bior_same_variant   /Data/catalogs/1000_genomes/20130502_GRCh37/variants_nodups.v1/ALL.wgs.sites.vcf.tsv.bgz
Clinvar_20160515_GRCh37 bior_same_variant   /Data/catalogs/ClinVar/20160515_GRCh37/variants_nodups.v1/macarthur-lab_xml_txt.tsv.bgz
dbSNP139    bior_overlap    /Data/catalogs/dbSNP/139/chr17_GRCh37.tsv.bgz
dbSNP_142_GRCh37p13 bior_overlap    /Data/catalogs/dbSNP/142_GRCh37.p13/variants_nodups.v1/chr17.vcf.tsv.bgz
ESP_V2_GRCh37   bior_same_variant   /Data/catalogs/ESP/V2_GRCh37/variants.nodups.v1/ESP6500.vcf.tsv.bgz
```

The drill file tells `bior_annotate.sh` what fields need to be extracted from the sources listed in the catalog file.  

 * column 1 is the ShortUniqueName in the *datasource.properties file for that catalog
 * column 2 is what features you want to drill out of that catalog

Create your own drill file for the annotation sets you just downloaded by copying & pasting this information:

```
ExAC_r03_GRCh37_nodups  INFO.AC_NFE,INFO.AC
1000genomes_20130502_GRCh37_nodups  INFO.EUR_AF
Clinvar_20160515_GRCh37 pathogenic,hgvs_c,hgvs_p
dbSNP_142_GRCh37p13 INFO.PMC
ESP_V2_GRCh37   INFO.EA_AC,ALL._maf
```
You do not need to drill out information from all the catalogs in your catalog file.  For example, even though there is a catalog (`dbSNP139`) listed in the catalog file, I don't want any of that information extracted since I have a newer version of dbSNP available(`dbSNP_142_GRCh37p13`). 
The name in the catalog_file must match the name in the drill_file EXACTLY.         

> Important: Make sure you have these tab seperated instead of spaces!

#### Download the `bior_annoate.sh` [Docker](https://www.docker.com/) Image from [Dockerhub](https://hub.docker.com/r/stevenhart/bior_annotate/)
This will take several minutes.

```
docker run -it --rm stevenhart/bior_annotate:latest bior_annotate.sh -h
```
You'll see the help information as soon as it completes downloading.

#### Run the demo
Use the `test.vcf` that you downloaded from before.

```
docker run -it --rm  -v $PWD:/Data -w /Data stevenhart/bior_annotate bior_annotate.sh -v test.vcf -o OUT -c catalog.file -d drill.file  
```
We need to mount the local directory into a directory called `Data` in the container (that's the docker `-v $PWD:/Data` parameter) and set the intial container working directory to `Data` (docker `-w /Data` paramter).

#### Now check out your results
You should now have 2 output files:
 * `OUT.vcf.gz`: The compressed & annotated VCF
 * `OUT.vcf.gz.tbi`: The index file for the compressed VCF

# All done!
