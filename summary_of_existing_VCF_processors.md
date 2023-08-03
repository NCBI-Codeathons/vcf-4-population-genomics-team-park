# Summary of the existing VCF operation tools


## VCF creating tools (BAM -> VCF, e.g. variant callers)
  - Population variant callers — key thing is that it is quite common in human genetics to do joint variant calling (the pipeline contains a per-sample processing step and then a whole-population joint calling step), meaning that for large populations, it is likely that GATK itself will need to natively write the new-VCF format.
    - GATK HaplotypeCaller and Mutect2 (See [Best Practices Workflow](https://gatk.broadinstitute.org/hc/en-us/sections/360007226651-Best-Practices-Workflows))
    - Google DeepVariant
  - Single sample variant calling and/or intrahost variant calling — may not need to natively support the new format, can get away with calling a conversion tool afterwards.
    - [freebayes](https://github.com/freebayes/freebayes)
    - [lofreq](https://github.com/CSB5/lofreq) – for intrahost variant calling
    - [ivar](https://github.com/andersen-lab/ivar) variants – particularly for viral
    - quasitools hydra – particularly HIV
    - bcftools mpileup/call
  - Population data (MSA fasta -> VCF) — would likely need a replacement implementation to natively write new-VCF format (and function at this scale).
    - UCSC faToVcf
    - Sanger’s [snp-sites](https://github.com/sanger-pathogens/snp-sites)

 ## VCF transforming tools (VCF->VCF, e.g. annotation, filtering)
  - [ANNOVAR](https://annovar.openbioinformatics.org/en/latest/)
    - a command-line tool for annotating genetic variants from VCF files. 
    - It takes VCF files as input and provides functional annotations, population frequencies, and pathogenicity predictions.
  - [VEP](https://useast.ensembl.org/info/docs/tools/vep/index.html)
    - an Ensembl tool used for annotating and predicting the functional consequences of variants in VCF files. 
    - It takes VCF files as input and provides valuable information about the impact of variants on genes and their potential effects.
  - GATK has some filtering and annotation functionality
  - Bcftools has some filtering and querying functionality
  - [SnpEff/SnpSift](http://pcingola.github.io/SnpEff/) - annotation and filter

## VCF consuming tools (VCF -> something else / insights)
  - Consensus callers (VCF -> FASTA) — these are single-sample inputs and may be okay with conversion tools / not native support
    - bcftools consensus
    - (ivar consensus does *not* use VCF as input, it goes straight from BAM)
  - Phylogenetic tools (VCF -> trees)
    - Iqtree, fasttree typically start with msa fasta inputs, but can take phylip distance matrixes instead — if we produce a new-VCF to phylip conversion tool, that solves it
    - Augur wrappers on iqtree/fasttree can use VCF input instead of fasta, but use custom code to convert
    - USHER uses VCF inputs for incremental daily builds of SC2 trees (adding 10k-100k genomes at a time), but cannot use VCF for full builds (VCF cannot hold 10M columns). Custom code for reading VCF.
 - Visual interactive analysis tools (VCF -> GUI)
   - [IGV](https://software.broadinstitute.org/software/igv/)
   - [3DVizSNP](https://analysistools.cancer.gov/3dvizsnp/about)
   - [Geneious](https://www.geneious.com/)
 - [GEMINI](https://gemini.readthedocs.io/en/latest/)
   - GEMINI is a flexible and powerful CLI tool set that allows querying and analysis of VCF files for genetic variation data, including annotations and population frequencies.
 - [PLINK](https://www.cog-genomics.org/plink/2.0/) (GWAS)
   - PLINK 2.0 has added support for VCF files as input
   - See notes on PGEN data format
