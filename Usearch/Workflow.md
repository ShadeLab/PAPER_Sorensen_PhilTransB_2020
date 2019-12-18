# Process for cutting adaptors/primers from sequences

## Listing all the samples
`ls *L001_R*.fastq > SampleNames.txt`
## Clipping pieces of samples name for easier use in loop
`sed -i 's/R1_001.fastq//g' SampleNames.txt`

## Make Directory for Trimmed Reads
` mkdir Trimmed_Reads`

## Loop for removing reads with primers
for file in $(<SamplesNames.txt)
do
cutadapt --discard -b GTGCCAGCMGCCGCGGTAA -G GGACTACHVGGGTWTCTAAT -o Trimmed_Reads/${file}trimmed_R1.fastq -p Trimmed_Reads/${file}trimmed_R2.fastq ${file}R1_001.fastq ${file}R2_001.fastq
done

## Merge fastq Reads from all samples
```
for file in $(<SampleNames.txt)
do
/mnt/research/rdp/public/thirdParty/usearch11.0.667_i86linux64 -fastq_mergepairs Trimmed_Reads/${file}trimmed_R1.fastq -reverse Trimmed_Reads/${file}trimmed_R2.fastq -fastqout Merged_Reads/${file}merged.fastq -fastq_pctid 85 -sample ${file}

done

cat Merged_Reads/* > Merged_Reads/combined_merged.fastq
```

## Dereplicate sequences
```
usearch64 -fastx_uniques combined_merged.fastq -fastqout uniques_combined_merged_trimmed.fastq -sizeout
```

## Cluster OTUs at 97% Identity
```
usearch64 -cluster_otus uniques_combined_merged.fastq -otus combined_merged_otus.fa -uparseout combined_merged_otus_uparse.txt -relabel OTU

```

## Map reads at 97% to 97% OTUs
```
usearch64 -otutab combined_merged.fastq -otus combined_merged_otus.fa -uc map_combined_merged_otus.uc -otutabout table_combined_merged_trimmed_otus.txt -biomout table_combined_merged_trimmed_otus.biom -notmatchedfq unmapped_combined_merged_trimmed.fq
```

## Classify 97% OTUs using sintax and Silva123
```
usearch64 -singtax Traditional_OTU/combined_merged_otus.fa -db silva_16S_v123.fa -tabbedout Traditional_OTU/combined_merged_both_runs_otus_taxonomy.sintax -strand both"
```