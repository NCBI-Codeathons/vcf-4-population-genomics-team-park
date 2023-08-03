# Summary of Proposed Alternatives to VCF Files
## OpenCGA
### Overview:
OpenCGA is a project aimed at developing an open-source framework for storing and analyzing genomic data at large scale (hundreds of terabytes to petabytes).  It uses NoSQL databases to hold genomic data and Apache Hadoop for data processing and storage.  They use standardized schema to represent biological data in their databases and provide interfaces to read VCF files into their database formats.
### Analysis:
OpenGCA seems less of a replacement for VCF files and more of a tool to allow better or higher-performance searching of variant data than is feasible with a text file implementation.  The tool seems to still be under development, or, at least, the documentation has large missing sections.  My skim over the documentation suggests that OpenGCA does not currently support modifying databases well.  It seems more intended to use input VCF files as the source of data and provide a front-end for querying the data those files contain.

Integrating a new variant file format with OpenGCA seems relatively easy.  All that would be required is a tool to read files in the new format into their database format.  If OpenGCA is extended to better support modifying its databases, for example by allowing incremental variant files to be added to a database, tools to write their databases back out to VCF or other formats would be useful.
## Scalable Variant Call Representation (SVCR)
### Overview:
SVCR is a proposal for a data model, rather than a file format, intended to represent large genomic datasets.  The document is a bit unclear what they mean by this. One of the figures shows a SVCR database of that is more than 50TB in size, so I assume that SVCR supports storing data on disk and not just in memory.  

The key difference between SVCR and VCF is that SVCR is a sparse format that only represents the regions where each sample differs from the reference genome.  In contrast, VCF is a dense format, with an entry for every sample at every position where at least one sample varies from the reference genome.  This makes SVCR representations much smaller than VCF representations in the (common) case where many of the samples only vary from the reference genome at a fraction of the locations where at least one sample varies from the reference.

The sparse representation also makes it easy to merge in data that varies from the reference genome in a new position. While adding such a sample to a VCF file would require creating a new row with entries for all of the existing samples showing that they do not vary from the reference genome, SVCR can simply add entries for the samples that do vary at that position.

One downside to SVCR is that locating the information for a given location of variance can be harder than in VCF, since the entries for each location vary in size.

### Analysis:
SVCR definitely seems to reduce the size of variance files, with one example showing a 3x reduction in size.  It is also a lossless representation, in that one could generate a VCF file from it.

The difficulty of supporting SVCR seems to me like it depends heavily on how the program uses its data files.  Programs that read their data files into a representation in memory would just require a reader for SVCR files.  Programs that access the data files dynamically at runtime, which strikes me as the common case given the size of the example files, would need a replacement library that replicates all the functions that the program uses.

One note is that SVCR seems much more optimized for row-wise access (comparing how each of the samples varies or does not vary from the reference at a particular position) than column-wise access (seeing all of the variants in a given sample).  This will affect which applications this format is a good match for.

## Sparse Allele Vectors
### Overview:
Like SVCR, Sparse Allele Vectors (SAV) reduce the size of variant data by using a sparse representation that only contains data for samples that vary at a given position.  It is a direct extension of the BCF variant of VCF files that adds a sparse row representation to the file format.  It also supports positional Burrows-Wheeler transform and Zstandard compression to further reduce file size.  Their paper reports similar file size reduction to SVCR.
### Analysis:
Conceptually, SAV is a direct extension to BCF, but the file formats appear fairly different.  SAV uses different compression algorithms than BCF, and uses a sort-tile-recursive r-tree to index the data file in order to reduce the time required to access random rows of data.  Thus, there would be little or no ability to re-use BCF parsers for SAV files.  However, the similar organization of the data suggests that implementing an SAV-compatible library that provided the same functionality as a VCF-parsing library might be relatively easy.  Again, as this is a sparse format, operations on rows of data are likely to be easier than operations on columns.

## Sparse Project VCF
### Overview:
Sparse Project VCF (spVCF) reduces the file size of variant data via two techniques.  The first, which they call "squeezing," discards most of the quality control (QC) data for cells where zero reads differed from the reference genotype.  In addition to reducing the amount of data in the file, this reduces the entropy of the QC data, improving the performance of traditional compression techniques.  However, it discards information, making it impossible to reproduce the original variant data from a spVCF file.

The second technique reduces the amount of space taken up by cells with identical data.  Cells that do not encode variations from the reference genome and whose contents are identical to the cell above them in the VCF table are replaced with a quotation mark, and then run-length encoding is applied to each row to further compress the data.  This step is lossless. One issue with this technique is that it could theoretically be necessary to look up an arbitrary number of rows to find the value of a cell that has been replaced with a quotation mark, making data extraction challenging.  To address this, the format periodically chooses some rows as "checkpoints" and represents them in their uncompressed form, thus limiting the amount of look-back required to de-compress a row.

### Analysis:
Because decompressing a row of data generally requires information from some of the rows above it, this format seems best-suited to applications that either want to process an entire variant data file at a times or to extract individual columns (samples) of data.  This file format does not seem to include any index for the data records, making access to an individual row of data slow.  Because of this, replicating the functionality of VCF-reading libraries on spVCF might be challenging.

It does deliver better file compression than many of its competitors, generally reducing file size by 14-15x, so might be a good format for archival storage of variant data if the loss of QC information is acceptable.  (I.e., store data in spVCF format when not being accessed and convert individual data files to some more easily used format before doing analysis on them.)

## Hail
### Overview:
It is an open source library that analyses tabular and matrix data variable of the GWAS dataset, with the ability to query and manipulate specific sections of the data. This is a Python tool, with the installation requiring Java, C++, Python 3.8 libraries preinstalled. This tool requires a VCF file for importing the data in the Hail MatrixTable format in the python environment, making it easier to run downstream analysis in python. The row field is used for storing the gene information alongwith information of the structural variants in the array format while the column field is for storing the phenotypic and metadata information. The expression data are present in the cells. The GWAS analysis can be performed using statistical tools such as Matplotlib, Pandas and the linear regression function wherein the covariates can be annotated with respect to the phenotypes. We can even cluster for genotypes in the samples using PCA. It can compute the read depth per variant per sample. Also, it has a built-in wrapper for variant effect predictor.

### Analysis:
Hail is pretty much a scalable genomic analysis tool instead of a vcf replacement, its advantages are that it offers fast variant calling analysis using this tool and the matrixtable is also accessible for performing further analysis. It runs on python environment. But it still requires vcf input, thereby defeating the purpose of being an alternative to vcf file format.

## GenomicsDB
### Overview:
It is a scalable WGS data storage tool. It can be used to store variant data like an array and the data can be queried and manipulated. It is written in C++ format and can be run on Linux shell. It is packages into GATK4 workflow and one if its key features is that it can merge gVCF files for joint genotyping. GenomicsDB uses a storage system for optimized storing/querying of sparse arrays, thus saving lots of space in the memory. Or else, it can also be used to perform genotypic analysis using multisample file or genomic databases workspace simultaneously using CombineGVCFs, with the caveat being that the combined files being made with CombineGVCFs function. Other programs might not give accurate results.

### Analysis:
Again, like Hail, it is just a format to store the variant data, and use it for efficient downstream analysis. It is not a separate file format, although it provides the advantage of merging multiple VCF files and can be used at a population level genomic analysis.

## GDS
### Overview:
This is a R/Bioconductor based tool, was mainly built to optimise large scale GWAS in R. It stores the Part of a project named CoreArray, having functions such as gdsfmt for efficient memory and file management, SNPRelate for GWAS SNP calculations on multicore systems. SNPRelate can also can be used for Plink, sequencevcf, netcdf, other data types with package to reconvert. The file is created in a .gds format. The .gds files contains genetic covariance and IBD (Identical by descent) coefficient split into non-overlapping parts and allocated to multicore system. The genome wide distribution of SNP effects attributed to allelic dosages and sample SNP eigenvectors, which helps to calculate substantial relatedness. The IBD estimation done by method of moment(mom) or maximum liklehood estimate(MLE).

There are 4 genotypes assigned for each byte, leading to faster computation and parallelization wherein 4 genotypes are being studied at once. Data blocking also helps so that only subset of data can be selected for processing, efficient access of large dataset. The genotypes are loaded block by block, limited by the size of main memory.

### Analysis:
This is a very efficient format which can help compress data with the help of calculating the covaraiance matrix of the SNP, and indexing these covaraints for saving memory.

## Tachylon
### Overview:
Open source C++ tool for calling variant data and writing, manipulating over it. It is also used for storing the SNP data in .yon format. It can be used for fast analysis on population level genomics data. This format stores data that optimises query execution, just like .cram does for .sam/.bam files. It can be downloaded in Linux using c library htslib as prerequisite, along with openssl, zstd for compression. While compressing, we need to set the block size of no. of variants/ no. of base pairs. It also uses encryption key using AES-256 to protect sensitive information. And just like Bcftools, it can be used to read/mutate the variant data file, alongwith the annotation of the metadata file.

### Analysis:
Its main function is that it helps in storage of population genomic level data, since it saves a lot of memory by storing variant data in columnar format i.e. stores necessary variant data in columns, while compressing the metadata to achieve more efficient storage of data with increase in sample size. Also, its encryption function can help protect the data.

## BGEN
### Overview:
It is a file format denoting the binary format of the genetic (.gen) file system. It also has the ability to impute data, store unphased genotype and phased haplotype data. It converts the data into small file size, through efficient bit representation and compression. The BGEN tool is written in C++, which can be used for .bgen format support in other softwares. BGEN saves memory by storing the allelic probabilities per sample, in a 3D array format. These probabilities are indexed according to each variant in the row, and the sample id and the ploidy for each sample at each variant. Unused genotype probability with no variants are marked as NA’s. Many .bgen files can also be concatenated using the cat-bgen function, and edited using edit-bgen. 

### Analysis:
This format helps to save a lot of memory by this method, but the support for more complex conditions are still under development and will not be compatible with this format.

## Genotype Query Tools (GQT)
### Overview:
GQT lies somewhere between an alternate file format for variant data and an application that uses variant data.  It's goal is to provide efficient extraction of an individual's variant data and comparison of variant data between individuals or groups of individuals.  The target application is disease/medical research, for example trying to understand what variations from the reference genome are associated with individuals who exhibit certain symptoms or respond in a particular way to medication.

GQT first transposes the array of information in a VCF file to create an array with one row per individual sampled and one column per location of variation.  It then sorts the columns of the transposed array by number of individuals who vary at that position and then converts each cell of the resulting matrix to four bits of data that show the nucleotide the individual has at that position.  The result is a very sparse array with long stretches of zeroes that compresses very well.  They also have a technique to add indices to their compressed data that make it easy to extract all of the individuals that have a particular attribute.

### Analysis:
GQT discards so much information that it is not a viable replacement for VCF.  It is more of an intermediate format that VCF data can be transformed into to support particular types of queries.  The tools that the authors provide to manipulate GQT files rely on htslib to read input VCF files, so an htslib-compatible library to access a new variant data file format would make it easy to port the GQT tools to support the new file format.

## Zarr:

## Overview:
Zarr is a more-generic data representation/format that has been adapted to handle variant data.  Zarr natively supports large N-dimenniosal arrays, with chunking and compression to reduce size and move array data to/from disk.  Scikit-allel and sgkit support storing variant data in Zarr, with each field in the VCF file represented as a separate Zarr array.  Storing variant data in Zarr seems to reduce file size by about 50% compared to BCF/VCF.  One advantage of Zarr over many of its competitors is that supports analysis of both rows (variance at a given position) and columns (individual data) well.
### Analysis:
A system based on Zarr seems like it would have potential as a replacement for VCF.  One potential disadvantage is that the Zarr representation creates a complex file structure for each set of variant data that would have to be transmitted as a tarfile, Zip archive, or similar object to make moving variant data from location to location practical.  Because the Zarr format is so different from VCF, writing work-alike libraries to some of the ones that process VCF files seems like the best way to drive adoption of this format.

## Pgen (PLINK)

### Overview:
Plink is a tool for Whole genome analysis on a large scale, focussing on analysing genotypic/phenotypic data. The files are stored in .pgen file format which is a 
binary format of genetic data, allowing for compressed storage, with the SNPack style genomic compression, helping decrease filesize by 80+%. The genotypes are 
represented as binary digits along with the allele frequencies. It also contains metadata like sample ids, marker ids,etc. It also has some support for visualization.

### Analysis:
It is similar to the bgen file format with the added feature of being compatible with Plink. The disadvantage though is that there is no focus on study design and 
planning, generating cnv calling while using this file format. Also, Plink can only work on unphased biallelic data, while bgen is capable of working with a wider
range of genetic data types.

## VCF Local Alleles:

## Overview:
In the context of VCF local alleles, the alleles are specifically selected and localised and the analysis done is selective for the given variant locus. This can help write large datasets into less data, thereby reducing the size of vcf's generated from SNP calling for as large as >20000 samples by being specific and incorporating necessary parameters.

### Analysis:
In the HTS-lib documentation, it states that this feature is already incorporated for vcf version 4.3. So this is a feature already existing in the current vcf environment to reduce the file size for large datasets. But with the drawback of being restricted to localised alleles only instead for the variants.
