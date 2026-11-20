# claude_output


## Prompt 1 ##
 Create a local document that summarises the steps taken in the investigation in the drop in indel precision in WES runs. This document will contain 3 sections.

Section 1: a summary of the investigation and it's findings. Focus on the information contained within pdf "O-260716 TWE investigation run-191126-092906.pdf", but use the other pdf files and pptx documents to build an understanding of the background. The summary produced should be concise

Section 2: A summary of any correlating QC trends. Using the verity database, which is accessible through a worker URL in DNAnexus project 003_eggd_verity_dev, review the data for any QC trends from tools other than hap.py that correlate with the decrease in Indel Precision. The file ID of the verity database is file-JFy551Q4JKFVPKkJ7bqJ497k on DNAnexus and is held in 003_eggd_verity_dev.

Section 3: Mechanism proposal. The initial investigation suggests that a number of false positives and false negatives come from variants being called as the wrong genotype, for example being called as heterozygous rather than homozygous. Propose a mechanism that make have caused this to occur during NGS sequencing and create a list of possible root causes of the indel precision decrease

Make this document locally, do not make any edits to files used as background information for creating this document.
  
- produced  TWE_indel_precision_investigation_summary.md
- produced fig 1,2,3

## Prompt 2 ##

The first production run to show the decrease in indel precision is 260218_A01295_0701_AHCMLYDMX2, which was performed prior to the use of multiplexing in production runs (though multiplexing was in use for the multiplexing validation runs). Details of the multiplexing validation have been provided in the PDF file "DV-Multiplexing S1 flow cell QC assessment-191126-124411.pdf". Create more figures to show QC trends, using the same style as fig1_precision_vs_freemix.png, highlighting where changes are present in run  260218_A01295_0701_AHCMLYDMX2 as well as later runs with low indel precision.
  
- produced fig 4,5,6

## Prompt 3 ##

Update TWE_indel_precision_investigation_summary.md to reflect new findings
- updated TWE_indel_precision_investigation_summary.md
