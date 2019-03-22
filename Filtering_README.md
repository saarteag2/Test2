# Annotate Transmission Summary

These scripts enable the annotation and subsequent filtering of transmission summary files. There are two types of transmission summary files: flat (one line per sample-variant) and family aggregation (one line per variant-family). Analysis-based filtering options described here: https://docs.google.com/spreadsheets/d/1d28ojG4sLbMGYldkZrDMTaLf2rylM4_cCL7uvEEzZPo/edit?usp=sharing.

## Guideline
1. Main functions of these scripts
2. Main scripts
3. Overview of analysis based filtering script
4. Additional support scripts
5. Instructions for adding a new project
6. Hints for adding a new column

## 1. Main functions of these scripts:
1. **MAKE SHARDS**: To improve compute performance, split large transmission summary files into smaller shards
2. **ANNOTATION**: Annotate flat and family aggregation transmission summary files with critical information (e.g., Genotype , GIAB region, HGNC name)
3. **ANALYSIS BASED FILTERING OF A FILTERED FILE**: Not parallelized. Only recommended for filtering a smaller pre-filtered file (no shards or per chr features built in)
4. **ANALYSIS BASED FILTERING**: Filter annotated flat and family aggregation transmission summary files from shards and recombine shards into one file. 

## 2. Main scripts:
1. **MAKE SHARDS**: To improve compute performance, split large transmission summary files into smaller shards
    1. /ifs/collabs/geschwind/bmap_scripts/process_vcf_line_by_line/split_and_gzip.py #source code to split
    2. qsub_split_gz_ms2.sh #qsub submission code example. User must make a project-specific version of a script. 

2. **ANNOTATION**: Annotate flat and family aggregation transmission summary files with critical information (e.g., Genotype, GIAB region, HGNC name)
    1. annotate_flat_transmission_summary.py #annotates flat files
    2. annotate_family_aggregation_transmission_summary.py #annotates family aggregation files
    3. qsub_shards_iHART.sh #qsub submission code (flat & family) example. User must make a project-specific version of a script. 

3. **ANALYSIS BASED FILTERING OF A FILTERED FILE**: Not parallelized. Only recommended for filtering a smaller pre-filtered file (no shards or per chr features built in)
    1. filter_flat_annotated_transmission_summary.py
    2. filter_family_aggregation_annotated_transmission_summary.py

4. **ANALYSIS BASED FILTERING**: Filter annotated flat and family aggregation transmission summary files from shards and recombine shards into one file.
    1. filter_flat_annotated_transmission_summary.py #source code to submit
    2. filter_family_aggregation_annotated_transmission_summary.py #source code to submit
    3. filter_annotated_transmission_summary_qsub.sh #Most efficent for first large filtering step. 
    3b. combine_and_clean.sh #source code at the end of filter_annotated_transmission_summary_qsub.sh to give you a single filtered output file.

## 3. Overview of analysis based filtering scripts:
Filtering can be conducted for the two types of transmission summary files: flat (filter_flat_annotated_transmission_summary.py) and family aggregation (filter_family_aggregation_annotated_transmission_summary.py). Both script have a set of required arguments and noteworthy arguments:

#### Required Arguments:
  1. **--input_db**: absolute path to input file (annotated flat file)
  2. **--output_dir**: absolute path to desired output directory
  3. **--input_type**: specify transmission summary input format as either: per_chr, multi_chr, or chr_shards (see details below)<br/>
Requirment dependent on use of --output_prefix parameter (see Noteworthy Arguments below)
  4. **--output_prefix**: desired output file prefix 

#### Noteworthy Arguments:
  1. **--comb_and_clean**: specify outfile format as either: merge_all_chr (Default), merge_by_chr, no_comb_clean
  2. **--no_qsub**: specify whether qsub can be used to filter multi-chr transmission summary input format type: False (Default), True

Both scripts accept transmission summary files in three formats that have been categorized into two categories: per_chr and multi_chr (see below for details). 
![Transmission Summary Input formats and input_type](https://i.imgur.com/Xndwp9u.png)

Input in multi_chr format are preprocessed by the script and divided into per chromosome shards in a directory called "filtered_flat_per_chr_shards" in the user specified output directory (see below). By default, filtering of these per chromosome shards are submitted as qsub jobs. One can opt out of using qsub, by providing --no_qsub True as an argument.
![Preprocessing Step](https://i.imgur.com/k7YDWhy.png)

Both scripts also allow the user to specify in which format they would like their filtering output in by using the --comb_and_clean argument to select: merge_all_chr (Default), merge_by_chr, or no_comb_clean (see details below). If the --comb_and_clean is specified as anything but no_comb_clean, then the --output_perfix argument must be set as well (see Required Arguments above).
![Combine and Clean Options](https://i.imgur.com/BrzX9lP.png)

## 4. Additional support scripts:
1. get_cohort_stats/get_flat_cohort_stats.py #Function? 
2. giab_dict_performance_test/ #A set of tests done to optimize performance for GIAB annotations?
3. line_counts #Old? Should be removed? 
4. add_gnomad_annotation_with_compatibility.py #Helpful script for annotating a flat file with gnomAD AFs. 
5. fats.py #Function? Used in filter_flat_annotated_transmission_summary.py
6. jobs_to_rerun.txt #Old? Should be removed?
7. qsub_patch_missing.sh #Function? 
8. qsub_patch_missing_family.sh #Function? 
9. qsub_tabix.sh #Makes .gz and .tbi VCF files to save space and improve speed for annotation/filtering.
10. summarize_annotated_transmission_summary_files_by_project.R #Summarizes flat and family aggregate transmission summary format per project 
11. divide_chr.py #Script to divide input db into per chromosome shards
12. divide_regions.py #Script to divide regions file into per chromosome files

## 5. Instructions for adding a new project:
1. Set project information in the script (currently starting on line #69): filter_annotated_transmission_summary_qsub.sh with a project name, directory_full_path, log_dir, and file_trunk. Add project name to Required arguments description for --project (currently line #6) and the Error line (currently line #95). 
2. Add project information (directory_full_path, file_trunk, and output_filename) to combine_and_clean.sh
3. Add project name to "Required_Parameters" on the google doc: https://docs.google.com/spreadsheets/d/1d28ojG4sLbMGYldkZrDMTaLf2rylM4_cCL7uvEEzZPo/edit?usp=sharing
4. Make a version of qsub_split_gz_ms2.sh, make project-specific edits, and submit this to split large transmission summary files. 
5. Make a version of qsub_shards_iHART.sh, make project-specific edits, and submit this to annotate transmission summary shards.
6. NOTE: If the split files now span a large genomic range (as is the case for a custom capture project) you may hit memory errors, in which you can request all 23G, you could add a parameter like -l h_vmem=23G to lines 38 and 46 in qsub_shards_NeuroReg.sh.

## 6. Hints for adding a new column:
1. If a new column is added it will go at the end of the columns (for consistency with other scripts)
2. Adding a new column into the annotated transmission summary file is relatively straightforward. Though you have to remember to add the code for adding a new column to both annotate_flat_transmission_summary.py and  annotate_family_aggregation_transmission_summary.py if desired.
3. Column names must be added to annotate_transmission_summary/fats.py in the static lists TransmissionSummaryLine.COLUMNS and FamilyAggregationTransmissionSummaryLine.FAMILY_AGGREGATION_COLUMNS if applicable (as well as TransmissionSummaryLine.NUMERIC_COLUMNS and FamilyAggregationTransmissionSummaryLine.NUMERIC_COLUMNS if the column should be treated as a numeric value). Otherwise you will have an error the first time you try to filter.

##### Documentation drafted by E.Ruzzo 2018-06-14, edited by S.Arteaga 2019-03-12.
