# SomaticSeq

* SomaticSeq is an ensemble caller that has the ability to use machine learning to filter out false positives. The detailed documentation is included in the package, located in [docs/Manual.pdf](docs/Manual.pdf "User Manual"). A quick guide can also be found [here](http://bioinform.github.io/somaticseq/).
* SomaticSeq's open-access paper: [Fang LT, Afshar PT, Chhibber A, et al. An ensemble approach to accurately detect somatic mutations using SomaticSeq. Genome Biol. 2015;16:197](http://dx.doi.org/10.1186/s13059-015-0758-2 "Fang LT, Afshar PT, Chhibber A, et al. An ensemble approach to accurately detect somatic mutations using SomaticSeq. Genome Biol. 2015;16:197.").
* Feel free to report issues and/or ask questions at the [Issues](../../issues "Issues") page. You may also email Li Tai Fang at [li_tai.fang@roche.com](li_tai.fang@roche.com).

## Requirements
* Python 3, plus regex, pysam, numpy, and scipy libraries
* R, plus [ada](https://cran.r-project.org/package=ada) library
* BEDTools
* Optional: dbSNP VCF file (if you want to use dbSNP membership as a feature)
* At least one of the callers we have incorporated, i.e., MuTect2/MuTect/Indelocator, VarScan2, JointSNVMix2, SomaticSniper, VarDict, MuSE, LoFreq, Scalpel, Strelka2, and/or TNscope.

## Example commands
* The following is a SomaticSeq command **after** the individual mutation caller jobs are complete
* If you're searching for pipelines to run those individual somatic mutation callers, feel free to take advantage of our [dockerized somatic mutation scripts](utilities/dockered_pipelines).

```
$somaticseq/SomaticSeq.Wrapper.sh \
--output-dir       /PATH/TO/RESULTS/SomaticSeq_MVSDULPK \
--genome-reference /PATH/TO/GRCh38.fa \
--tumor-bam        /PATH/TO/HCC1395.bam \
--normal-bam       /PATH/TO/HCC1395BL.bam \
--dbsnp            /PATH/TO/dbSNP.GRCh38.vcf \
--cosmic           /PATH/TO/COSMIC.v77.vcf \
--mutect2          /PATH/TO/RESULTS/MuTect2.vcf \
--varscan-snv      /PATH/TO/RESULTS/VarScan2.snp.vcf \
--varscan-indel    /PATH/TO/RESULTS/VarScan2.indel.vcf \
--sniper           /PATH/TO/RESULTS/SomaticSniper.vcf \
--vardict          /PATH/TO/RESULTS/VarDict.vcf \
--muse             /PATH/TO/RESULTS/MuSE.vcf \
--lofreq-snv       /PATH/TO/RESULTS/LoFreq.somatic_final.snvs.vcf.gz \
--lofreq-indel     /PATH/TO/RESULTS/LoFreq.somatic_final.indels.vcf.gz \
--scalpel          /PATH/TO/RESULTS/Scalpel.vcf \
--strelka-snv      /PATH/TO/RESULTS/Strelka/results/variants/somatic.snvs.vcf.gz \
--strelka-indel    /PATH/TO/RESULTS/Strelka/results/variants/somatic.indels.vcf.gz \
--inclusion-region /PATH/TO/RESULTS/captureRegion.bed \
--exclusion-region /PATH/TO/RESULTS/blackList.bed
```

* For all those input VCF files, either .vcf or .vcf.gz are acceptable. 

### Additional parameters for training/prediction:

    --truth-snv:        if you have ground truth VCF file for SNV
    --truth-indel:      if you have a ground truth VCF file for INDEL
    --ada-r-script:     $somaticseq/r_scripts/ada_model_builder_ntChange.R to build classifiers (.RData files), if you have ground truths supplied.
    --classifier-snv:   classifier (.RData file) previously built for SNV
    --classifier-indel: classifier (.RData file) previously built for INDEL
    --ada-r-script:     $somaticseq/r_scripts/ada_model_predictor.R to use the classifiers specified above to make predictions


* Do not worry if Python throws the following warning. This occurs when SciPy attempts a statistical test with empty data, e.g., z-scores between reference- and variant-supporting reads will be NaN if there is no reference read at a position.

   ```
     RuntimeWarning: invalid value encountered in double_scalars
     z = (s - expected) / np.sqrt(n1*n2*(n1+n2+1)/12.0)
   ```

## Dockerized applications and pipelines

### To run somatic mutation callers
We have created scripts that run all the dockerized somatic mutation callers and SomaticSeq at [**utilities/dockered_pipelines**](utilities/dockered_pipelines). All you need is [docker](https://www.docker.com/). 

### To create training data set
We have also dockerized pipelines for *in silico* mutation spike in at [**utilities/dockered_pipelines/bamSimulator**](utilities/dockered_pipelines/bamSimulator). 
These pipelines are based on [BAMSurgeon](https://github.com/adamewing/bamsurgeon). We use it to create training set to build SomaticSeq classifiers.

### GATK's best practices
The limited pipeline to generate BAM files based on GATK's best practices is at [utilities/dockered_pipelines/alignments](utilities/dockered_pipelines/alignments).

### Additional workflows
* A [Snakemake](https://snakemake.readthedocs.io/en/latest/) workflow to run the somatic mutation callers and SomaticSeq, created by [Afif Elghraoui](https://github.com/0xaf1f), is at [**utilities/snakemake**](utilities/snakemake).
* All the docker scripts have their corresponding singularity versions at utilities/singularities. They're created automatically with this [script](utilities/singularities/docker2singularity.py). They are not as extensively tested or optimized as the dockered ones. Read the pages at the dockered pipelines for descriptions and how-to's. Please let us know at Issues if any of them does not work.


## Video tutorial

This 8-minute video was created for SomaticSeq v1. The details are slightly outdated, but the main points remain the same. 

  [![SomaticSeq Video](docs/SomaticSeqYoutube.png)](https://www.youtube.com/watch?v=MnJdTQWWN6w "SomaticSeq Video")
