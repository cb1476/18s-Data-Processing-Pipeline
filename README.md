# 18S Metabarcoding in Wild Canid Respiratory Microbiome

## Group Members
Collin Blake

## Background
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;My data is not linked to a published research paper -- it comes from an ongoing research project seeking to characterize the respiratory microbiome of canids (wild and domestic) presenting with respiratory disease. A proposal for the study was submitted to the American Kennel Club, who has agreed to sponsor the project. The principal investigator for this study is David Needle, DVM DVCAP, and the running title for the proposal is "Characterizing potential novel CIRD pathogen and CIRD microbiome perturbations." Currently, the main focus of the research team is to characterize a novel respiratory mycoplasma suspected to induce respiratory disease in domestic canines; in the proposal, the primary pathogen of interest is referred to as "BARDID" (Bacteria Associated with Respiratory Disease In Dogs). Aside from characterizing BARDID, a secondary goal of the research team is to broadly describe the respiratory microbiome in canids; to this end, they have performed 16s and 18s barcoding to understand the composition of the respiratory microbiome of cainds. The majority of samples being tested are from domestic canines (canis familiaris), but respiratory tissue samples from the NHVDL biobank have also been included in order to explore the respiratory microbiomes of other canids -- specifically red foxes and grey foxes (Vulpes vulpes and Urocyon cinereoargenteus, respectively). My dataset includes multiplexed paired-end fastqs for the 18S barcodes of eukaryotic microbiota in 6 wild foxes and 1 domestic dog for which bulk tissue DNA extraction has been performed, followed by PCR amplification with the V4 18s primer and sequencing in the Illumina NovaSeq 6000 instrument with a read length of 250bp. The goal of my study is to analyze the multiplexed fastq files gained from 18S metabarcoding to identify the eukaryotic microbiota in wild canids (specifically red foxes and grey foxes). I will discuss discuss patterns in co-ocurrence of specific eukaryotic taxa within the respiratory microbiomes of canids whose tissues underwent 18S metabarcoding. From such data, I can speculate on how the co-presence of certain taxa within single hosts could affect both host immune responses and impact host mortality. There is no scientific literature (that I could find) on 18S metabarcoding in wild canids, so the data I'll present will hopefully provide new insights on the eukaryotic species that could contribute to respiratory disease in foxes from New Hampshire.
 

## Methods:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Drylab analyses were performed in GitBash on the Ron bioinformatics server hosted by the Hubbard Center for Genome Studies. The multiplexed sequence data from 250bp Illumina sequencing was imported into GitBash, and access for the data was provided by Dr. Joseph Sevingy from the Genomics Center. After data import, generation of a plot displaying the percent-reads mapped to each eukaryotic taxonomic class per-sample was accomplished via creation of a shell script that utilized tools/plugins from version 2024.5 of the QIIME2 bioinformatics platform. Scripting procedures and their rationale are listed as follows:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The qiime2 import tool was utilized to reformat the multiplexed sequence data (initially in fastq.qz format) into the appropriate qiime2 format (.qza) with a flag designating the original data type (i.e., paired end sequences with quality scores). The sequences of the forward and reverse primers for the 18s-V4 primer were defined as the variables “fw” and “rv” respectively. Following this, the CutAdapt tool was used to discard reads that didn’t contain the primer sequences in order to remove any adaptor contamination from PCR steps. Additionally, reads that contained the V4 primer sequence within the following specifications had their primers trimmed and were subsequently kept; 1) an error value of 0.12 was specified to allow a semi-conservative degree of mismatching (12%) between amplicon sequences and the target forward and reverse primer sequences; 2) ambiguous bases (i.e., IUPAC wildcards) within primer sequences were specified as keepable via addition of the “--p-match-adapter-wildcards” flag. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Denoising was performed with DADA2 for the purposes of error correction and chimeric removal, merging of paired-end reads, the production of a feature table listing the counts of ASVs in each sample, and a rep-seqs table listing the denoised, merged sequences that were the best-representatives for each ASV. Both forward and reverse reads were conservatively truncated to a length of 225 base pairs. The “pooling-method 'pseudo'” flag was included for two reasons: firstly, it is more appropriately-applied towards datasets for a relatively small number of samples – in this case, a dataset of barcodes for only seven samples; secondly, it has the added benefit of initiating the performance of initial independent denoising for each sample followed by subsequent ASV recording succeeded by a secondary round of denoising with a now-established “knowledge” of the prior-recorded ASVs so as to increase sensitivity to them (Callahan, 2025). 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Following this, the “extract-reads” tool in qiime2 was used to acquire the regions of the full-length 18S rRNA genes that specifically matched the V4-18s primers; these specific regions were extracted from the SILVA reference database, containing 18S rRNA gene sequences for a large range of organisms. A minimum identity of 0.85 (or 85%) between primer and reference sequences was specified to allow for a degree of flexibility that could account for sequencing errors, slight differences between primers, and biological variability. Then, a Naïve-Bayes classifier was trained on the 18S rRNA sequences (extracted from the SILVA database) that matched the 18S-V4 primer region – the purpose of which was to assign taxonomic labels to the ASVs from the wild canid dataset. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A two-stage hybrid approach was utilized to assign each ASV to a taxonomic category using the classify-hybrid-vsearch-sklearn tool in qiime2. A query-coverage of 0.8 was specified to ensure that during global alignment (in VSEARCH), 80% of the total length of each ASV needed to align to reference sequences to be considered an adequate match. Additional parameters included the following: a percent-identity of 0.9, specifying 90% total sequence similarity between the ASV and reference sequence; a “maxaccepts” value of 10, allowing for the creation of a consensus taxonomy that included the top ten best-aligned reference sequences for each ASV; conservatively, a minimum consensus value of 0.51 to ensure taxonomic assignment of each ASV to be for the genus which comprised >51% of the ten best-aligned reference sequences; and lastly, a minimum confidence threshold of 0.7 to ensure that the Naïve Bayes classifier would omit taxonomic classifications of an ASV at any taxonomic rank below the rank for which it was less than 70% confident in reporting. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The aforementioned minimum confidence threshold of 0.7 proved to be too narrow of a parameter, as initial test-runs of the script resulted in unclassified ASVs which prevented the creation of a comprehensive taxonomic bar plot. As such, it was necessary to rerun the alignment step at a lower minimum confidence threshold. To do so, the “qiime-taxa filter-table tool” was used to create a feature table that only included the ASVs that were classified at the initial confidence threshold of 0.7, removing all ASVs which were unassigned. Then, using a manually-created text file containing all ASVs that did not get assigned a taxon (104 in total), the “filter-seqs” tool was used to filter the original rep-seqs file (containing all denoised ASVs) for the purpose of generating a new rep-seqs file containing only the unclassified ASVs. The “unclassified rep-seqs” file was then utilized in a secondary run of hybrid alignment –this time with a minimum confidence threshold of 0.5. This new confidence threshold proved adequate enough to allow for complete taxonomic classification of all previously-unassigned ASVs. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The taxonomy table of ASVs assigned at a confidence threshold of 0.5 was merged with the taxonomy table from the original alignment run (at a confidence threshold of 0.7) with the “qiime2 rescript merge-taxa” tool to generate a merged taxonomy table for all ASVs. Following this, the “qiime taxa filter-table” plugin was utilized with the inclusion of the “--p-exclude Chordata,Vertebra” flag; given the nature of the samples (i.e., bulk tissue from canids), it was necessary to remove any ASVs mapped to the 18S barcodes from the canids (vertebrates) so as to only observe the barcodes from the microbiota (invertebrates). Lastly, the “qiime taxa barplot” tool was utilized to create the taxonomic barplot (in .qza format) – the input for which was the feature table (filtered for vertebrates), merged taxonomy table of ASVs assigned at confidence =0.5 and =0.7, and tabulated metadata file containing relevant sample data. I viewed the taxonomic barplot in the qiime2 viewer website, and I edited the barplot in a separate photo-editing app to display the actual species for each sample rather than their sample-IDs which were displayed by default.  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Following this, a secondary python script was generated to produce an UpSet plot displaying the percentage of the sample pool for which each detectible taxonomic group was present in. The python script required its input files to be in .tsv format; therefore, the filtered feature table from the earlier-described shell script was converted into a BIOM file initially, and then into a binned .tsv file – both of which required the use of the “qiime tools export” command. An abridged overview of the UpSet plot’s python script is provided below:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Various python libraries were imported for the purposes of file manipulation, data organization, plotting, and input-handling; the most relevant of these libraries were “pandas” and “matplotlib” as well as the “upsetplot” package. The “contents_dict” command was used to create an empty python dictionary that stored the binned Sample-IDs for which a specific taxon was present. The “bin_dict DictReader” command was used to open and read the tabulated feature table (specified as the first system argument) before converting it from .tsv format into the python dictionary format, which specifies the headers of rows as taxons and gives a list of the sample-IDs for which the taxon was observed in (regardless of the number of reads per-sample that mapped to the taxon). An example of the format conversion is given below: 

<img src="https://github.com/user-attachments/assets/a98cb98f-95bc-4e4d-a15a-05e2eb12be98" alt="Description" width="300" height="180"/>
<img src="https://github.com/user-attachments/assets/967d442d-05a1-427f-9378-d4d0d2937601" alt="Description" width="300" height="180"/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Once in the python dictionary format, a “for/if” loop was scripted to populate the empty contents dictionary with the data from the feature table. The way this worked is that the loop went through every row of the input file (i.e., “for row in bin_dict”), and if the column header was “BIN,” its “count” value was treated as the name of the bin (i.e., the taxon associated with the ASV of interest), and if the column’s header was not “BIN” then its count was validated to be greater than zero (i.e., the taxon was present in the sample) and the sample-ID was added to the contents dictionary under the “BIN” (the taxon) for which the sample possessed at least one ASV for. Then, the populated contents dictionary was converted to a format compatible with the UpSet format using the “from_contents()” function from the “upsetplot” package. The “matplotlib.rcParams” command from the matplot library was used to set the font size for the UpSet plot, and finally the “ax_dict = UpSet(<parameters>)” command was used to generate the UpSet plot within a set of parameters that allowed for better visualization of overlap between samples for shared taxa. 

### Tools:
#### “Qiime tools import”
&nbsp;&nbsp;&nbsp;&nbsp;This step of the pipeline was included to convert the paired-end fastq files into a qiime artifact (in .qza format). 

#### CutAdapt: “qiime cutadapt trim-paired”
&nbsp;&nbsp;&nbsp;&nbsp;Using my “fw” and “rv” variables defined in a previous step, I used the CutAdapt tool to trim primers from the reads that had them, and for reads that didn’t have the primers, the tool discarded them to ensure that only properly-amplified 18s barcodes were retained for use in later steps of the pipeline

#### DADA2: “qiime dada2 denoise-paired”
&nbsp;&nbsp;&nbsp;&nbsp;Denoising with DADA2 was done for the purposes of error correction and chimeric removal, merging of paired-end reads, the production of a feature table listing the counts of ASVs in each sample, and a rep-seqs table listing the denoised, merged sequences that were the best-representatives for each ASV.

#### “qiime feature-classifier extract-reads”
&nbsp;&nbsp;&nbsp;&nbsp;This tool was used to acquire the regions of the full-length 18S rRNA genes that specifically matched the V4-18s primers; these specific regions were extracted from the SILVA reference database, containing 18S rRNA gene sequences for a large range of organisms. 

#### “qiime feature-classifier fit-classifier-naive-bayes”
&nbsp;&nbsp;&nbsp;&nbsp;This tool was used to train a naive bayes classifier on my reference sequences and taxonomy, which was necessary for later steps of the pipeline involving alignment of the ASV to the correct taxa

#### “qiime feature-classifier classify-hybrid-vsearch-sklearn”
&nbsp;&nbsp;&nbsp;&nbsp;This tool was used for the purpose of classifying the rep-seqs (from denoising) using a hybrid classifier that classified sequences using a global alignment tool (Vsearch) and a machine learning program (sklearn). I used the hybrid classifier for my *first* round of feature classification, but because not all of my ASVs mapped to a taxa given the strict parameters set for the hybrid alignment tool, I opted to use the regular, non-hybrid “qiime feature-classifier classify-sklearn” tool for my second round of feature classification (and I used a lower confidence threshold in the second round as well). 

#### “qiime taxa filter-table”
&nbsp;&nbsp;&nbsp;&nbsp;I couldn’t initially make a barplot given how my first round of alignment missed some of the ASVs within my feature table, so I used this tool to filter the feature table so that it only included the ASVs that were classified at the initial confidence threshold of 0.7, removing all ASVs which were unassigned

#### “qiime feature-table summarize”
&nbsp;&nbsp;&nbsp;&nbsp;Like I said, I had initial trouble with getting all my ASVs classified the first time around, so I used this tool just as a troubleshooting step in order to do a quality assessment of my denoising output. I was mostly just interested in getting an idea of the number of ASVs that I was dealing with in order to see the proportion of my total amount of ASVs that failed to get classified in the previous steps of the pipeline.

#### “qiime metadata tabulate”
&nbsp;&nbsp;&nbsp;&nbsp;Again, another troubleshooting tool that I used to create a table of ASVs which DID get assigned a taxon, since not all of them did. Like the previous tool, I was able to view the output (a .qzv file) in the online qiime2 viewer. From viewing the outputs of this tool and the last tool in the qiime2 viewer (and from viewing the outputs of various error messages on the command line), I was able to very painfully (manually) create a text file that contained ALL of the feature-IDs for the ASVs that didn’t get classified in the first round of alignment. 

#### “qiime feature-table filter-seqs”
&nbsp;&nbsp;&nbsp;&nbsp;Using the aforementioned text file of ASVs that didn’t get classified in the first round of alignment, I used this tool to filter the original rep-seqs file from the denoising step in order to pull out only the rep-seqs for those ASVs that didn’t get classified. This new rep-seqs file was what I ran the second round of alignment on. 

#### “qiime rescript merge-taxa”
&nbsp;&nbsp;&nbsp;&nbsp;After the second round of alignment, I used this tool to merge the taxonomy tables from the first and second rounds of alignment to generate the final/complete taxonomy table containing the taxa that mapped to all of the feature-IDs for all of my ASVs.  

#### “qiime taxa filter-table”
&nbsp;&nbsp;&nbsp;&nbsp;I used this tool to filter out all vertebrates from my merged taxonomy table because this study only focused on finding the microbes (invertebrates) among the 7 tissue samples that underwent sequencing. 

#### “qiime taxa barplot”
&nbsp;&nbsp;&nbsp;&nbsp;This tool created a taxonomic barplot (a .qzv file) which I viewed in the qiime2 viewer. 

### Extra: The following tools were used to get my feature table into .tsv format from its original format (.qza) because that was the format needed to run the python script for my UpSet plot. Note: I put these commands into the script I used to make by barplot because I wasn’t sure if they would work in my python script:

#### “qiime tools export”
&nbsp;&nbsp;&nbsp;&nbsp;I used this tool to export the feature table (initially in .qza format) as a BIOM file

#### “biom convert”
&nbsp;&nbsp;&nbsp;&nbsp;I used this tool to convert my BIOM feature table file into .tsv format using the previously-generated “feature-table.biom” file as the input. I couldn’t find a way to directly convert my .qza file into a .tsv file so this step was necessary. 



## Findings
<img src="https://github.com/user-attachments/assets/cc8970d9-6114-4b84-b157-889026245706" alt="Description" width="300" height="300"/>
<img src="https://github.com/user-attachments/assets/bc6ca20c-2bb6-4f81-aac7-6d8e47c356f2" alt="Description" width="400" height="300"/>

#### Figure 1: Taxonomic barplot with legend. 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This barplot shows the %-reads per-taxa found in each of the seven samples. The legend is sorted to display (in descending order) each taxa for which the highest %-reads were displayed for all reads across all samples. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The input for this plot was the feature table (filtered for vertebrates), the merged taxonomy table of ASVs assigned at confidence =0.5 and =0.7, and tabulated metadata file containing relevant sample data. To get both the feature table and taxonomy table (and to run the barplot-creation command), a shell script was made an run in GitBash (as described in the methods section). I viewed the .qzv file (barplot output) in the qiime2 viewer and sorted it to display 6th level taxonomy so that the legend wouldn't be too crowded.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Plot interpretation: It seems that the taxa for which there were the most reads mapped to the _Rhabditida_ genus. In the qiime2 viewer, if I change the taxonomy level from 6th (seen in the barplot above) to 7th-level taxonomy (displays species), it can be seen that the specific _Rhabditida_ species was _Crenosoma vulpes_, or "Fox Lungworm." However, given that the left-most sample (one of the red foxes) had an incredibly high %-reads for this species, I would say that the results are skewed in favor towards this one taxa, so that taxa was not significantly present in most samples, only in one. The most significantly occurring taxa seem to be _Babesia_ (specifically _Babesia microti_) and _Hepatozoon_ (specifically _Hepatozoon canis_), both of which are single-celled, tick-borne protist parasites. Looking at the relative %-reads for either of the two parasite species (uppermost purple bars for _Babesia_, uppermost orange bars for _Hepatozoon_) reveal that their %-reads per-sample seem to be inversely proportional (i.e., when there is high %-babesia, there is low %-hepatozoon, and vice-versa). This could be because both species inhabit different cell types (_Babesia_ infects red blood cells, _hepatozoon_ infects white blood cells), so due to the fact that these cells could be present in different concentrations in the bulk tissue samples, it's more-likely that the inverse relationship between _Babesia_ and _Hepatozoon_ %-reads is just due to chance and not something else (i.e., intra-host competition between the two species). 



<img src="https://github.com/user-attachments/assets/2c775bf0-d440-4c9f-bae1-64e2cd835d88" alt="Description" width="300" height="400"/>

#### Figure 2: UpSet plot 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This UpSet plot does a better job at showing co-occurence of taxa between samples, but it does not take into account the %-reads for each taxa. So, for example, for a sample with 80% of its ASVs mapping to taxon A, it is treated as the same as a sample with 5% of its ASVs mapping to taxon A (because both samples contained at least one ASV for taxon A). 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The input for this plot was the feature table from the initial barplot script (converted from .qza into .tsv format), and the command used to run the python script for the UpSet plot was: "python upset_plot.py /home/users/cb1476/exported_feature_table/feature-table.tsv /home/users/cb1476/upset_plot.png". Note that output for the command was a png file, so there was no need to view it in the qiime2 viewer. 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Plot interpretation: The plot shows that the following taxa were present at any detectible level in more than one of the samples from the collective sample pool: Malassezia –  a genus of commensal skin yeasts (6/7 samples); Babesia – a genus of tick-borne, single-celled intraerythrocytic parasites that can cause hemolytic anemia (5/7 samples); Hepatozoon – a genus of tick-borne, single-celled parasitic protists that primarily infect hemolymph and leukocytes as well as tissues of the liver and spleen (5/7 samples); Pneumocystis – single-celled ascomycetes that, while normally commensal, can cause pneumocystis pneumonia (4/7 samples); Rhabditida – multi-cellular free-living nematodes that can parasitize plants and animals (3/7 samples), Magnoliophyta – a clade of flowering plants (3/7 samples), Toxoplasma – genus containing one species, Toxoplasma gondii, for which there are multiple haplogroups which all infect warm-blooded animals (2/7 samples). 

## References
Needle, David B. “Characterizing Potential Novel CIRD Pathogen and CIRD Microbiome Perturbations.” American Kennel Club. 29 Feb. 2024. 

Bokulich, Nicholas A., et al. “Optimizing Taxonomic Classification of Marker-Gene Amplicon Sequences with QIIME 2’s Q2-Feature-Classifier Plugin.” Microbiome, vol. 6, no. 1, 2018, p. 90, doi:10.1186/s40168-018-0470-z.

Bolyen, Evan, et al. “Reproducible, Interactive, Scalable and Extensible Microbiome Data Science Using QIIME 2.” Nature Biotechnology, vol. 37, no. 8, 2019, pp. 852–57, doi:10.1038/s41587-019-0209-9.
---. “Reproducible, Interactive, Scalable and Extensible Microbiome Data Science Using QIIME 2.” Nature Biotechnology, vol. 37, no. 8, 2019, pp. 852–57, doi:10.1038/s41587-019-0209-9.

Callahan, Benjamin J., et al. “DADA2: High-Resolution Sample Inference from Illumina Amplicon Data.” Nature Methods, vol. 13, no. 7, Nature Publishing Group, 2016, p. 581, doi:10.1038/nmeth.3869.

Martin, Marcel. “Cutadapt Removes Adapter Sequences from High-Throughput Sequencing Reads.” EMBnet. Journal, vol. 17, no. 1, 2011, p. pp-10, doi:10.14806/ej.17.1.200.

McDonald, Daniel, et al. “The Biological Observation Matrix (BIOM) Format or: How I Learned to Stop Worrying and Love the Ome-Ome.” GigaScience, vol. 1, no. 1, BioMed Central, 2012, p. 7, doi:10.1186/2047-217X-1-7.
---. “The Biological Observation Matrix (BIOM) Format or: How I Learned to Stop Worrying and Love the Ome-Ome.” GigaScience, vol. 1, no. 1, BioMed Central, 2012, p. 7, doi:10.1186/2047-217X-1-7.

Pedregosa, Fabian, et al. “Scikit-Learn: Machine Learning in Python.” Journal of Machine Learning Research, vol. 12, no. Oct, 2011, pp. 2825–30.
---. “Scikit-Learn: Machine Learning in Python.” Journal of Machine Learning Research, vol. 12, no. Oct, 2011, pp. 2825–30.

Pruesse, Elmar, et al. “SILVA: A Comprehensive Online Resource for Quality Checked and Aligned Ribosomal RNA Sequence Data Compatible with ARB.” Nucleic Acids Res, vol. 35, no. 21, 2007, pp. 7188–96.
---. “SILVA: A Comprehensive Online Resource for Quality Checked and Aligned Ribosomal RNA Sequence Data Compatible with ARB.” Nucleic Acids Res, vol. 35, no. 21, 2007, pp. 7188–96.

Quast, Christian, et al. “The SILVA Ribosomal RNA Gene Database Project: Improved Data Processing and Web-Based Tools.” Nucleic Acids Res, vol. 41, no. Database issue, Oxford University Press, 2013, pp. D590-6.
---. “The SILVA Ribosomal RNA Gene Database Project: Improved Data Processing and Web-Based Tools.” Nucleic Acids Res, vol. 41, no. Database issue, Oxford University Press, 2013, pp. D590-6.

Robeson, Michael S., et al. “RESCRIPt: Reproducible Sequence Taxonomy Reference Database Management.” PLoS Computational Biology, Public Library of Science, 2021, doi:10.1371/journal.pcbi.1009581.

Rognes, Torbjørn, et al. “VSEARCH: A Versatile Open Source Tool for Metagenomics.” PeerJ, vol. 4, PeerJ Inc., 2016, p. e2584, doi:10.7717/peerj.2584.

Wes McKinney. “ Data Structures for Statistical Computing in Python .” Proceedings of the 9th Python in Science Conference , edited by Stéfan van der Walt and Jarrod Millman, 2010, pp. 51–56.
---. “ Data Structures for Statistical Computing in Python .” Proceedings of the 9th Python in Science Conference , edited by Stéfan van der Walt and Jarrod Millman, 2010, pp. 51–56.
---. “ Data Structures for Statistical Computing in Python .” Proceedings of the 9th Python in Science Conference , edited by Stéfan van der Walt and Jarrod Millman, 2010, pp. 51–56.
---. “ Data Structures for Statistical Computing in Python .” Proceedings of the 9th Python in Science Conference , edited by Stéfan van der Walt and Jarrod Millman, 2010, pp. 51–56.
