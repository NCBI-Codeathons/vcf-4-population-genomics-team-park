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

