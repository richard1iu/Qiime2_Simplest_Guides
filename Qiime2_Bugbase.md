# Qiime2Bugbase
## 1. Import 99_otus_fasta into qiime2
>Import rep_set/99_otus_fasta of gg_13_5 **(bugbase requires 13-5)** into qiime2
```
qiime tools import --input-path ~/gg_13_5_otus/rep_set/99_otus.fasta --output-path gg_99_otus.qza --type FeatureData[Sequence]
```
## 2.Vsearch cluster
>Input file:
1. rep-seqs-dada2.qza from dada2
2. table-dada2.qza output from dada2
3. gg_99_otus.qza from our step 1

```
qiime vsearch cluster-features-closed-reference \
--i-sequences rep-seqs-dada2.qza \
--i-table table-dada2.qza \
--i-reference-sequences gg_99_otus.qza \
--p-perc-identity 0.97 \
--o-clustered-table table-cr-99.qza \
--o-clustered-sequences rep-seqs-cr-99.qza \
--o-unmatched-sequences unmatched-seqs \
--verbose
```

## 3.Format conversion
Input file of bugbase is **biom 1.0 JSON**，and the colname should be "taxonomy" instead of "OTU ID"  
so we need 
1) extract otutab from biom (output biom of qiime2 is html format) and  
2) change colname "OTU ID" to "taxonomy"
```
# Extract feature-table.biom
qiime tools export \
--input-path table-cr-99.qza \
--output-path $PWD
```
from the step above we got the file feature-table.biom, but its colname is "OTU ID".

## 4.Add colname for file 99_otu_taxonomy.txt (#OTUID，taxonomy）
```
echo -e "#OTUID\ttaxonomy" | cat - 99_otu_taxonomy.txt > 99_otu_taxonomy.txt
```

## 5.Add the column taxonomy to biom
```
biom add-metadata -i feature-table.biom -o feature-table-tax.biom --observation-metadata-fp 99_otu_taxonomy.txt --sc-separated taxonomy
```

## 6.Change the biom2.0 file to biom1.0（JSON）：
1. convert biom2.0 to txt
```
biom convert --table-type="OTU table" -i feature-table-tax.biom -o feature-table-tax.txt --to-tsv --header-key taxonomy
```
2. convert txt to biom1.0
```
biom convert -i feature-table-tax.txt -o feature-table-tax-biom1.biom --table-type="OTU table" --to-json --process-obs-metadata taxonomy
```

## 7.convert the colname of metadata to "#SampleID"
