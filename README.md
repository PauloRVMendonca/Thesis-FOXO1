# Thesis-FOXO1
Code used for analysis in Python
Key steps performed in this work. RNA-seq data from normal and FOXO1 overexpressing HUVECs were analysed in this work. (1) alignment of reads (Kallisto and STAR), and building of transcripts through de novo assembly (StringTie); (2) comparison of transcriptomes of normal HUVECs and FOXO1 overexpressing HUVECs (GFFCompare); (3) identification of alternative splicing (AS) events (Vast-tools); (4) differential expression analysis of transcripts and genes (edgeR); (5) identification of isoforms switch between normal and FOXO1 overexpressing HUVECs; and (6) intersection of the results of all methods of RNA-seq analysis used to identify the genes that might have suffer more changes between the two types of samples.

Commands used to generate genome index and aligning the reads to to the reference genome
(STAR alligner):
● Building index: udocker run -v=/$PWD/:/$PWD/ -w=/$PWD starf STAR --runThreadN $THREADS
--runMode genomeGenerate --genomeDir ./genome_dir --genomeFastaFiles
./genome_dir/genome.fa --sjdbGTFfile ./genome_dir/annotation.gtf --sjdbOverhang 100
● Mapping reads (sample SRR8758292): udocker -q run -v=/$PWD/:/$PWD/ -w=/$PWD starf STAR
\ --runThreadN $THREADS --genomeDir $GENOME_DIR --outFileNamePrefix
${OUT_DIR}${SAMPLE} --outSAMtype BAM SortedByCoordinate --readFilesIn
$INPUT_FASTQ --quantMode GeneCounts --sjdbGTFfile $GTF_FILE --outFilterMultimapNmax 1

Command used for assembly of the aligned reads (Stringtie):
● Transcriptome assembly: udocker -q run -v=/$PWD/:/$PWD/ -w=/$PWD stringtie stringtie -o
./output_dir/output.gtf -p 8 -A ./output_dir/gene_abund.tab --rf ./alignment_dir/input alligned.bam
Command used for comparison of GTF files obtained from assembly step against the reference genome (GFF compare wasn’t a docker image):
● GFFCompare utility: ./gffcompare -o output_prefix -r reference_annotation.gtf -R
assemble_annotation.gtf -D -S -C -A -X

Commands used for each steps required for alternative splicing analysis (Vast-tools):
● Allignment: for file in $(ls .(path to fastqz files of read)/*.fastq.gz); do udocker run -v=/$PWD/:/$PWD/ -w=/$PWD vasttools vast-tools align ${file} -dbDir /(path to Vast-tools database dir) --output (path to dir where to store output) -sp Hs2 -c 8 --expr; done
● Combine: udocker run -v=/$PWD/:/$PWD/ -w=/$PWD vasttools vast-tools combine -dbDir /(path to Vasttools database dir) --output (path to dir where to store output) -sp Hs2 -c 8 –C
● Diff: udocker run -v=/$PWD/:/$PWD/ -w=/$PWD vasttools vast-tools diff -i (path to inclusion tabs provided by vast-tools combine) -a (Replicates of FOXO1 overexpression) -b (Replicates of control) -o(path to dir where to store output) -c 8 -d diff_output --minReads=10 --minDiff=0.1

Commands used to generate kallisto index and aligning (Kallisto installed):
● Building index: ./(path to where kallisto tool)/kallisto index -i index_file transcriptome_file(provided from Ensemble https://github.com/pachterlab/kallisto-transcriptome-indices)
● Alignment: for file in $(ls (path to reads dir) *.fastq.gz); do (path to kallisto tool)/kallisto quant --rf-stranded -i (path to index) -o (path to output dir)/${file} --single -l 200 -s 20 -t 8 $file; done
