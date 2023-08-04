# Preparing for a new VCF standard

List of participants and affiliations:
- Daniel Park, Broad Institute (Team Leader)
- Nick Carter, Harvard University (Tech Lead)
- Yixing Han, NIH/NHGRI (Writer)
- Maria Burgos-Garay, CDC
- Devesh Haseja, University of Birmingham


## Project Goals
Given several ongoing projects for improved technical solutions under consideration to replace VCF, our group’s goal is to define a minimum set of bioinformatic tools that any solution must implement to simplify adoption in VCF-centric analysis pipelines. Our output will be a set of recommendations for those who are working on serialized data formats that improve upon VCF for storing, indexing, and searching population genetic data at large scale.

Our goals do not include: evaluating or selecting a preferred technical solution, or to implement any of these recommended changes ourselves. 

## Approach
VCF (or BCF) is a serialized and indexed data format for storing population genetic data that encounters problems at certain levels of scale—in particular, for very large populations. Several efforts exist to implement different data structures that solve some of the limitations of the current VCF format—these are each being utilized by different communities and use cases, but the GA4GH is also evaluating whether any of these should replace VCF more broadly.

Regardless of the technical merits of any of these proposed file formats, one of the biggest barriers to community adoption of a new file format is the required updates in existing analytic pipelines to each step that currently relies on VCF for its analyses. We will perform a landscape analysis of the types of analytic pipelines that interact with VCF and the common ways in which they both create and consume them. We will examine pipelines that are commonly used in viral, bacterial, and human genomic analyses. We will identify the common touch points with VCF files and identify a minimal set of bioinformatic tools that must be provided by an advocate of any proposed file format if they want to increase adoption. Such touch points may likely include:

Conversion to and from new file format to BCF/VCF/gVCF
  - It is likely that most simple variant callers (gatk, freebayes, ivar) would not need to support writing the new format directly (at least in the near term)--a conversion tool (from VCF to new format) could be just fine initially.
  
  - Implementation of some (or all?) of bcftools functionality for the new file format. Almost certainly “bcftools merge” at a minimum, but likely more (query, view, annotate, reheader, consensus, call).
    Or is it feasible/possible to simply extend/PR on htslib to support your new format and then bcftools (and pysam) would then naturally support it?
  
  - It’s possible that joint variant callers (e.g. GATK4) may need to be extended to write the new file format natively?

  - Implementation of a tool to produce a phylip-formatted distance matrix from the new file format (this would enable most phylogenetic software to produce trees from this representation, iqtree and fasttree can use phylip distance matrixes as input), something like vcf2phylip or snp-dists. Augur/treetime seems to prefer converting a VCF into a variants-only MSA fasta, but not sure if this is necessary.

In parallel, we will perform a high-level look through the proposed variant file formats to understand the commonality of the problems they attempt to solve, their approaches to solve them, and their assumptions about their use cases and how users interact with their data format. It is possible that we may need to refine our approach and outputs based on what we find here.

## Results
- [Analysis of proposed new data representations / VCF replacements](https://github.com/NCBI-Codeathons/vcf-4-population-genomics-team-park/blob/main/summary_of_alternatives.md)

- [Analysis of ecosystem of tools that create, consume and change VCF files](https://github.com/NCBI-Codeathons/vcf-4-population-genomics-team-park/edit/main/summary_of_existing_VCF_processors.md)

- [Final presentation (PPTX)](https://github.com/NCBI-Codeathons/vcf-4-population-genomics-team-park/blob/main/NCBI%20VCF%20Codeathon%20-%20Team%20Park.pptx)
