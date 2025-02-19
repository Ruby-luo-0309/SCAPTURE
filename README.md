# SCAPTURE
![image](https://github.com/YangLab/SCAPTURE/blob/main/scripts/Schematic%20diagram.jpg)


Description:
	SCAPRUTE pipeline. A deep learning-embedded pipeline that captures polyadenylation information from 3 prime tag-based RNA-seq of single cells

## Installation:
* Require [featureCounts ≥ v1.6.2](http://bioinf.wehi.edu.au/featureCounts/)

* Create conda env from [SCAPTURE yaml file](https://github.com/YangLab/SCAPTURE/blob/main/SCAPTURE_env.yml):

		conda env create -f SCAPTURE_env.yaml
	
* Download and unzip package from Github, then add "+x" to scripts to make it executable.
	
		unzip SCAPTURE-main.zip
		cd SCAPTURE-main/
		chmod +x scapture scapture_annotation scapture_callpeak scapture_evaluate scapture_mergesample scapture_quant

* Export path of SCAPTURE suite to environment

		SCAPTURE="/Your_Path/SCAPTURE-main"
	Add SCAPTURE to ./bashrc file of your environment.
	
		echo "export ${SCAPTURE}:\$PATH" >> ~/.bashrc
	Or manually export to environment variable before running SCAPTURE
	
		export PATH=${SCAPTURE}:$PATH
		
	Test whether SCAPTURE could be found
	
		which scapture
		"/Your_Path/SCAPTURE-main/scapture"
		
* Activate SCAPTURE_env environment before running SCAPTURE

		conda activate SCAPTURE_env
		
## Usage:

    scapture -m <annotation|PAScall|PASmerge|PASquant> [options]
             -h/---help  -- help information

### Module:         annotation

	Options:

	-o          -- prefix of the output files
	-g          -- genome .fa file
	--gtf       -- gene annotation file (GTF format, recomannd GENOCODE annotation with "gene_name" and "gene_type" tags)
	--cs        -- chromosom size file
	--extend    -- extended dowstream distance of 3 prime gene annotation to call peaks (bp, default: 2000)
	-h/---help  -- help information
	     
### Module:         PAScall

Options:

	-a          -- prefix of output annotation files in "annotation" module
	-o          -- prefix of output file
	-b          -- Cell Ranger aligned BAM file
	-g          -- genome .fa file
	-p          -- number of cores
	-l          -- read length of sequenceing cDNA in scRNA-seq
	-w          -- width of poly(A) peaks (default: 400)
 	--species   -- species for DeepPASS model ('human', 'mouse')
	--overlap   -- overlapped ratio of exonic peaks to merge (default: 0.5)
	--polyaDB   -- poly(A) site database bed6 file (optional)
	-h/---help  -- help information
 
### Module:         PASmerge

	Options:

	-o          -- prefix of output file
	--peak      -- list of evaluated peak files to merge 
	               (one sample per line, split by tab,
	               1st col: "Sample_name",
	               2cd col: "PathofEvaluatedPeakFile" )
	--path      -- path of scapture suite (ignore if scapture in PATH)
	--rawpeak   -- raw peak files to merge (Restricted by --peak)
	-h/---help  -- help information
 
### Module:         PASquant 

	Options:

	-o          -- output prefix
	-p          -- number of threads
	-b          -- Cell Ranger aligned BAM file
	--pas       -- PASs peaks file to quantify
	--celllist  -- cell barcode file as white list(one barcode per line),
	               or use "--celllist FromBAM" to used all cell barcode 
	               from input BAM (for unfiltered BAM, it might take lots of time).
	--celltype  -- cell type annotation file
	               (tab split, 
	               1st col: "cell_barcode",
	               2cd col: "cell_type" ).
	--bw        -- generate bigwiggle file for each
                cell type in --celltype option
                (true, false, default: false)
	-h/---help  -- help information

Version: 1.0 2021/01/25

Author: Guo-Wei Li Email: liguowei@picb.ac.cn

## Step-by-step protocol

  We use six human PBMC scRNA-seq datasets from 10x Genomics to show step-by-step protocol of SCAPTURE. All files used/generated by this example could be found at [shared Google Drive](https://drive.google.com/drive/folders/1XgC0MBYe4MBQn_lMtq0L9t7Dtv5oGK3P?usp=sharing).

### File preparation	
* Make sure that featureCounts and SCAPTURE are available in your PATH. User can manually add software path to PATH, like: 

		export PATH=/home/software/SCAPTURE-main:$PATH
		export PATH=/home/software/subread-1.6.2-Linux-x86_64/bin:$PATH

* Prepare gene annotation and genome file. In this example, we use annotation from [GENCODE hg38](https://www.gencodegenes.org/human/). SCAPTURE supports references given by users for specific study. Currently, we take GENCODE GTF file as gene annotation. GENCODE GTF annotation contains "gene_name" and "gene_type" in the 8th column field, and SCAPTURE utilizes the information to annotate PAS from genes. However, it is not necessary for the future (we are working on it).    

		gencode.v34.annotation.gtf
		hg38.chromsize
		genome.fa
		genome.fa.fai
		
* If poly(A) databases file (BED format, optional) is given, SCAPTURE will annotate PAS with database information and select relatively high confident PAS during merging different samples (in SCAPTURE PASmerge step). Combined PolyADB3, PolyA-Seq, PolyASite and GENCODE poly(A) databases file: [human hg38](https://drive.google.com/file/d/14ZQo9FWmMDRvIcR6mqXVMfIdQRKsaEL3/view?usp=sharing) (SupTab_KnownPASs_fourDBs.txt), [mouse mm10](https://drive.google.com/file/d/1UOorNJ63OJaenM6e7j6e0zFa_Q0uKZeO/view?usp=sharing) (mm10.SupTab_KnownPASs_fourDBs.txt).
	
		SupTab_KnownPASs_fourDBs.txt
	
* For running demo, we extracted reads in chr1:1-10000000 region from aligned BAM file of PBMCs in out study. And the white cell barcode list in each sample (generated by Cell Ranger count or given by the user). All used files could be found at [here](https://drive.google.com/drive/folders/1XgC0MBYe4MBQn_lMtq0L9t7Dtv5oGK3P?usp=sharing). 

		ls ./PBMC*
		PBMC10k.test.bam PBMC8k.test.bam PBMC6k.test.bam PBMC5k.test.bam PBMC4k.test.bam PBMC3k.test.bam
		PBMC10k.test.bam.bai PBMC8k.test.bam.bai PBMC6k.test.bam.bai PBMC5k.test.bam.bai PBMC4k.test.bam.bai PBMC3k.test.bam.bai
		PBMC10k.celllist PBMC8k.celllist PBMC6k.celllist PBMC5k.celllist PBMC4k.celllist PBMC3k.celllist

### Run scapture annotation module

  In this step, SCAPTURE will generate annotation files for downstream analysis. If your study includes multiple samples, we recommend using the same annotation files in this step to analyze all samples.

	scapture -m annotation -o SCAPTURE_annotation -g genome.fa --gtf hg38.gtf --cs hg38.chromsize --extend 2000 &> annotation.log

  Output files in this step:
  
	ls SCAPTURE_annotation*
	SCAPTURE_annotation.chromsize    SCAPTURE_annotation.genetype.bed           SCAPTURE_annotation.genetype.gtf
	SCAPTURE_annotation.element.txt  SCAPTURE_annotation.genetype.extended.bed  SCAPTURE_annotation.genetype.txt
	
### Run scapture PAScall module

  SCAPTURE takes aligned BAM files and above annotation files as input to call peaks. The PAScall module contains several steps:

  (1) perform transcript level peak calling and filtering
  
  (2) assign peak to APA-transcript (see detail in our paper)
	
  (3) evaluate peak with DeepPASS prediction (automatically run) and poly(A) database annotation (optional)
	
	scapture -m PAScall -a SCAPTURE_annotation -g genome.fa -b PBMC3k.test.bam  -l 98 -o PBMC3k -p 16 --species human --polyaDB SupTab_KnownPASs_fourDBs.txt &> PBMC3k.PAScall.log
	scapture -m PAScall -a SCAPTURE_annotation -g genome.fa -b PBMC4k.test.bam  -l 98 -o PBMC4k -p 16 --species human --polyaDB SupTab_KnownPASs_fourDBs.txt &> PBMC4k.PAScall.log
	scapture -m PAScall -a SCAPTURE_annotation -g genome.fa -b PBMC5k.test.bam  -l 91 -o PBMC5k -p 16 --species human --polyaDB SupTab_KnownPASs_fourDBs.txt &> PBMC5k.PAScall.log
	scapture -m PAScall -a SCAPTURE_annotation -g genome.fa -b PBMC6k.test.bam  -l 98 -o PBMC6k -p 16 --species human --polyaDB SupTab_KnownPASs_fourDBs.txt &> PBMC6k.PAScall.log
	scapture -m PAScall -a SCAPTURE_annotation -g genome.fa -b PBMC8k.test.bam  -l 98 -o PBMC8k -p 16 --species human --polyaDB SupTab_KnownPASs_fourDBs.txt &> PBMC8k.PAScall.log
	scapture -m PAScall -a SCAPTURE_annotation -g genome.fa -b PBMC10k.test.bam -l 91 -o PBMC10k -p 16 --species human --polyaDB SupTab_KnownPASs_fourDBs.txt &> PBMC10k.PAScall.log
	
  Output files in this step:
  
  	#for the PBMC3k sample, we got:
	PBMC3k.exonic.peaks.bed # filtered exonic peaks
	PBMC3k.exonic.peaks.annotated.bed # assigned exonic peaks
	PBMC3k.exonic.peaks.evaluated.bed # evaluated exonic peaks
	PBMC3k.intronic.peaks.bed # filtered intronic peaks
	PBMC3k.intronic.peaks.annotated.bed # assigned intronic peaks
	PBMC3k.intronic.peaks.evaluated.bed # evaluated intronic peaks
	PBMC3k.3primeExtended.peaks.annotated.bed # assigned peaks in dowstream region beyond 3' end of genes
	PBMC3k.3primeExtended.peaks.evaluated.bed # evaluated peaks in dowstream region beyond 3' end of genes

  The output poly(A) site file was in widely-used [BED format](https://genome.ucsc.edu/FAQ/FAQformat.html#format1.7) [see](https://github.com/YangLab/SCAPTURE/blob/main/scripts/BED12_format.png).
  
  Briefly, 
 * the 1-12 columns represent the spliced peak region of PAS.  [see](https://github.com/YangLab/SCAPTURE/blob/main/scripts/BED12_format.png)

  Specifically, for "evaluated.bed",
 * the 13th column is "number of refernce poly(A) sites supporting the PAS."
 * the 14th column is "DeepPASS prediction of the PAS."
 * the 15th column is "±100 nt sequence around the cleavage site of PAS."
	   

### Run scapture PASmerge module

If your study includes multiple samples, SCAPTURE can merge the peak result from all samples, and create a combined peak reference file for downstream analysis.

	#create table with sample name and sample peak file path:
	for i in 3 4 5 6 8 10; do echo -e PBMC${i}k"\t"PBMC${i}k.exonic.peaks.evaluated.bed; done > PBMC_ALL.exonic.peaklist
	for i in 3 4 5 6 8 10; do echo -e PBMC${i}k"\t"PBMC${i}k.intronic.peaks.evaluated.bed; done > PBMC_ALL.intronic.peaklist
	for i in 3 4 5 6 8 10; do echo -e PBMC${i}k"\t"PBMC${i}k.3primeExtended.peaks.evaluated.bed; done > PBMC_ALL.3primeExtended.peaklist

	#run the PASmerge module
	scapture -m PASmerge -o PBMC_ALL.exonic --peak PBMC_ALL.exonic.peaklist &> PBMC_ALL.PASmerge.exonic.log
	scapture -m PASmerge -o PBMC_ALL.intronic --peak PBMC_ALL.intronic.peaklist &> PBMC_ALL.PASmerge.intronic.log
	scapture -m PASmerge -o PBMC_ALL.3primeExtended --peak PBMC_ALL.3primeExtended.peaklist &> PBMC_ALL.PASmerge.3primeExtended.log

  Output files in this step:
  	
	# Integrated peaks
	PBMC_ALL.exonic.Integrated.bed
	PBMC_ALL.intronic.Integrated.bed
	PBMC_ALL.3primeExtended.Integrated.bed

	# Integrated peaks with original record of all samples
	PBMC_ALL.exonic.IntegratedSamples.bed
	PBMC_ALL.intronic.IntegratedSamples.bed
	PBMC_ALL.3primeExtended.IntegratedSamples.bed
	
  Combined peaks are written in files with "Integrated.bed" [BED format](https://genome.ucsc.edu/FAQ/FAQformat.html#format1.7) [see](https://github.com/YangLab/SCAPTURE/blob/main/scripts/BED12_format.png),
 * the 1-12 columns represent the spliced peak region of PAS [see](https://github.com/YangLab/SCAPTURE/blob/main/scripts/BED12_format.png). 
 * the 13th column is "number of refernce poly(A) sites supporting the PAS."
 * the 14th column is "DeepPASS prediction of the PAS."
 * the 15th column is "±100 nt sequence around the cleavage site of PAS."

  Specifically, for "IntegratedSamples.bed" (ignored, not used in downstream analysis),
 * the 16th column is the index id during peak combination.
 * the 17th column the number of peaks with the same index id.


### Run scapture PASquant module

  For PAS quantifying at single-cell level expression, users can chose different subset of PASs for their goals. In our paper, we recomanded select the high-cofident PASs with positive prediction and konwn sites overlapped.
  
	# 1. Select PASs with positive prediction and konwn sites overlapped (recomanded)
	perl -alne '$,="\t";print @F[0..11] if $F[12] > 0 | $F[13] eq "positive";' PBMC_ALL.exonic.Integrated.bed PBMC_ALL.intronic.Integrated.bed > PBMC_ALL.PASquant.bed
	
	# 2. Select PASs with positive prediction (no konwn sites filter)
	perl -alne '$,="\t";print @F[0..11] if $F[13] eq "positive";' PBMC_ALL.exonic.Integrated.bed PBMC_ALL.intronic.Integrated.bed > PBMC_ALL.PASquant.bed
	
	# 3. Select all raw PASs
	perl -alne '$,="\t";print @F[0..11];' PBMC_ALL.exonic.Integrated.bed PBMC_ALL.intronic.Integrated.bed > PBMC_ALL.PASquant.bed
	
	
  Quantify high-confidence PASs in six PBMC samples:
  
	scapture -m PASquant -b PBMC3k.test.bam --celllist PBMC3k.celllist --pas PBMC_ALL.PASquant.bed -o PBMC3k.PASquant &> PBMC3k.PASquant.log
	scapture -m PASquant -b PBMC4k.test.bam --celllist PBMC4k.celllist --pas PBMC_ALL.PASquant.bed -o PBMC4k.PASquant &> PBMC4k.PASquant.log
	scapture -m PASquant -b PBMC5k.test.bam --celllist PBMC5k.celllist --pas PBMC_ALL.PASquant.bed -o PBMC5k.PASquant &> PBMC5k.PASquant.log
	scapture -m PASquant -b PBMC6k.test.bam --celllist PBMC6k.celllist --pas PBMC_ALL.PASquant.bed -o PBMC6k.PASquant &> PBMC6k.PASquant.log
	scapture -m PASquant -b PBMC8k.test.bam --celllist PBMC8k.celllist --pas PBMC_ALL.PASquant.bed -o PBMC8k.PASquant &> PBMC8k.PASquant.log
	scapture -m PASquant -b PBMC10k.test.bam --celllist PBMC10k.celllist --pas PBMC_ALL.PASquant.bed -o PBMC10k.PASquant &> PBMC10k.PASquant.log
	
  Output files in this step:
  
	PBMC3k.PASquant.KeepCell.UMIs.tsv.gz
	PBMC4k.PASquant.KeepCell.UMIs.tsv.gz
	PBMC5k.PASquant.KeepCell.UMIs.tsv.gz
	PBMC6k.PASquant.KeepCell.UMIs.tsv.gz
	PBMC8k.PASquant.KeepCell.UMIs.tsv.gz
	PBMC10k.PASquant.KeepCell.UMIs.tsv.gz

# Single-cell APA analysis vignettes:
	
* A vignettes of single-cell APA analysis about COVID-19 study could be found at https://github.com/YangLab/SCAPTURE/blob/main/Vignettes.html.zip.


## License

  Copyright ©2021 Shanghai Institute of Nutrition and Health. All Rights Reserved.

  Licensed GPLv3 for open source use or contact YangLab (yanglab@picb.ac.cn) for commercial use.

  Permission to use, copy, modify, and distribute this software and its documentation for educational, research, and not-for-profit purposes, without fee and without a signed licensing agreement, is hereby granted, provided that the above copyright notice in all copies, modifications, and distributions. 

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
