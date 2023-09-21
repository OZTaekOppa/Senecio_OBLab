# Senecio_OBLab
##Senecio Transcriptome Part (Zoe)

##For FASTQC and Trimmomatic, please, see my previous training info that is already in the OB lab training GitHub page.

# Transcriptome assembly (Trinity)
##Contig assembly from RNA-Seq

Trinity --seqType fq --max_memory 400G  \
        --left /Filtered_UnZ_FQ/{SCAG_All.R1.fq, TSCGR_All.R1.fq} \
        --right /Filtered_UnZ_FQ/{ SCAG_All.R2.fq, SCGR_All.R2.fq } \
        --CPU 12
	      --min_contig_length 200
	      --full_cleanup


# Transcriptome contigs mapping (GMAP)
##Generate gff3

gmap -D /SL19Y_GMAP_DB -d SL19Y_GMAP_DB -t 8 -O AUnig_Ref.fasta -f 2 > GMAP_UnigRef_SL19Y_Out
gmap -D /SL19Y_GMAP_DB -d SL19Y_GMAP_DB -t 8 -O Trinity_ASM_MD.fasta -f 2 > GMAP_TrnASM_SL19Y_Out


# RNA-Seq mapping and TPM calculation (hisat2, samtools, tpmcalculator)
##HiSat2 Index (SL19PB Genome Reference)

hisat2-build -f /SL19_PBCtg.fasta SL19PB -p 4

##HiSat2 mapping

hisat2 -x /SL19PB \
-1 /Filtered_UnZ_FQ/P1_1_PETR1.fq \
-2 /Filtered_UnZ_FQ/P1_2_PETR2.fq -S SLP1.sam -p 8 

##Import SAM to BAM when SQ absent Test2 (using Fasta file to include headers)

samtools faidx /SL_GMAPDB/SL19_PBCtg.fasta
samtools view -bT /SL19_PBCtg.fasta SLP1.sam > SLP1.bam

##bam file coordiate sort ASC

samtools sort -@8 -T $TMPDIR/SLP1.sorted -O bam -o SLP1.sorted.bam SLP1.bam
samtools index SLP1.sorted.bam

##TPMCalculator currently works with RNA-seq reads mapped to the genome, where the gtf annotation is used to calculate the expression levels

TPMCalculator -g /SL_GMAPDB/UnigRef_SL19_Org.gtf \
-d /AgraVT/P1AG \
-b /AgraVT/P1AG/SLP1.sorted.bam

