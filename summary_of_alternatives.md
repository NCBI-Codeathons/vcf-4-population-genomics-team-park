# Summary of Proposed Alternatives to VCF Files

## OpenCGA
### Overview:
OpenCGA is a project aimed at developing an open-source framework for storing and analyzing genomic data at large scale (hundreds of terabytes to petabytes).  It uses NoSQL databases to hold genomic data and Apache Hadoop for data processing and storage.  They use standardized schema to represent biological data in their databases and provide interfaces to read VCF files into their database formats.

### Analysis:
OpenGCA seems less of a replacement for VCF files and more of a tool to allow better or higher-performance searching of variant data than is feasible with a text file implementation.  The tool seems to still be under development, or, at least, the documentation has large missing sections.  My skim over the documentation suggests that OpenGCA does not currently support modifying databases well.  It seems more intended to use input VCF files as the source of data and provide a front-end for querying the data those files contain.

Integrating a new variant file format with OpenGCA seems relatively easy.  All that would be required is a tool to read files in the new format into their database format.  If OpenGCA is extended to better support modifying its databases, for example by allowing incremental variant files to be added to a database, tools to write their databases back out to VCF or other formats would be useful.