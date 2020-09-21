Motif discovery within clusters of upstream sequences in plant genomes
======================================================================

i. Summary
----------

This protocol explains step-by-step how to discover DNA motifs in regulatory regions of clusters of co-expressed gene in plants. It also explains how to control the significance of the result, and how to annotate the discovered motifs with putative binding fators. It uses the plant-dedicated mirror of the Regulatory Sequence Analysis Tools (RSAT, <http://plants.rsat.eu>) but it can be used on any RSAT mirror.

1. Introduction
---------------

Transcriptome data (microarrays, RNA-seq) have been extensively used as a proxy for genetic regulation in many organisms, as the analysis of genome-wide profiles of gene transcription under different treatments uncovers clusters of genes with correlated behaviours, which may result from direct or indirect co-regulation. A classical application of this approach was done by Beer and co-workers (1) with yeast microarray data sets obtained in a variety of experimental conditions. In that experiment, expression data-mining was demonstrated to be an effective strategy for finding regulons, groups of genes that share regulatory mechanisms and functional annotations.

Other studies have unveiled that the outcome of these approaches largely depends on the genomic background of the species under study. For instance, Sand and others (2) reported that the significance of DNA motifs discovered in *Saccharomyces cerevisiae* promoters is much higher for regulons than for random gene sets of the same sizes, but for human promoters the signal-to-noise ratio is almost null, because random gene sets give highly significant motifs due to heterogeneities in promoter compositions and biases due to repetitive elements. For metazoans, it is thus a real challenge to distinguish *bona fide* motifs from noise (2). These observations suggest that motif discovery on sequence clusters faces intrinsic properties of the genomes under study, regardless of the software used for the task.

Among plants, these strategies have so far been tested on the model *Arabidopsis thaliana*, and they have been succesfully applied to the identification of novel cis regulatory elements validated with synthetic promoters (3). Yet, with the exception of this model, this sort of experiments have not been possible in plants until recently. In spite of this, the growing list of available plant genomes encourages these analyses in combination with expression profiles, as in the recent work of Yu and collaborators (4), provided that these factors are considered:

-   Plant genomes are rich in repetitive elements (RE) distributed along the genome (5), which pose particular problems for motif discovery statistics (violation of the independence assumption).

-   Current genome assemblies range from 119.7Mb (*A.thaliana*) to 14.54Gb (*Triticum aestivum*). *Brachypodium distachyon*, a model species for grasses, is 271.9Mb. The quality of these assemblies and their RE content is also quite variable (see stats [here](http://rsat.eead.csic.es/plants/data/stats/).

-   Upstream regions, defined by annotated gene coordinates, are also of variable length.

This tutorial is step-by-step protocol for the task of discovering and annotating DNA motifs in clusters of upstream sequences for species supported on the RSAT::Plants server, which have been obtained mostly from Ensembl Plants (<http://plants.ensembl.org>), but also include data from the JGI Genome Portal (<http://genome.jgi.doe.gov>) and the National Institute of Agrobiological Sciences in Japan (<http://barleyflc.dna.affrc.go.jp/bexdb/>). In addition, RSAT::Plants integrates footprintDB (<http://floresta.eead.csic.es/footprintdb>), a collection of position-specific scoring matrices (PSSM) representing transcription factor binding motifs (TFBM), as well as their cognate binding proteins (6), which can be used to annotate discovered motifs and to predict potentially binding transcription factors.

2. Materials
------------

This protocol requires to dispose of:

-   A computer with any Web browser installed.

-   A set of gene clusters from any of the species currently supported at RSAT::Plants ( <http://plants.rsat.eu>, see **Note **[1]). Here we will use three example clusters of maize genes, shown on **Table 2**, found to be co-expressed in 22 transcriptomes and enriched on some Gene Ontology terms according to their authors *(4)*. See **Note **[2]. **Table 3** counts the number of RNA-seq series available per taxa on database Gene Expression Omnibus <http://www.ncbi.nlm.nih.gov/geo>, and shows that plants currently represent ~10% of the datasets.

3. Methods
----------

<!-- This is the main section and should explain in detail the individual steps necessary to carry out the technique. Where possible, please simply list the steps in numerical order. For techniques that comprise a number of separate major procedures, please indicate these separate procedures in the introduction, and then subdivide section 3 into subheadings to cover each procedure (3.1, 3.2 etc). The steps in each subsection should then be numbered individually, renumbering  from number one. Do take great care to try to indicate any little "tricks" or nuances that help improve your method by referring to relevant "notes" in section 4 (see below). This sort of information rarely gets into the scientific literature. You may also find it useful to relate to some aspects of the theory in this section indicating the purpose of some of the major steps by cross-referencing to an appropriate “note”. Do not be tempted to get involved in the description of variations/alternatives to your technique in this section: this can be done in the "Notes" section. Stick to the basic procedure detailed in this section. This section must be comprehensive. Do not send the reader away to find information for a particular step in another reference. All relevant practical detail must be given in this section. -->
The following protocol enumerates the steps required to discover DNA motifs, based on the over-representation of k-mers (oligonucleotides) and dyads (spaced pairs of oligonucleotides), in clusters of upstream sequences. The protocol comprises two stages, analyzing first co-expressed genes and then random clusters as a negative control (see **Note **[3]). Only after both stages have been completed it is possible to objectively estimate the relevance of the results.

The time required for carrying out the following steps is approximately one hour.

<!-- ### Gene ontology enrichment of genes clusters 

This works on baobab:~rsat/marseille/protocolos/regulons/Zea_mays/go.mk, but has not been ported to the web yet:

retrieve-ensembl-go -org ${SPECIES} -i $@/regulon$@.txt -o $@/regulon$@.go
compare-classes -q $@/regulon$@.go -r ${REFCLASSES} -rnames ${GODESCRFILE} \
        -return rank,occ,proba,jac_sim -sort sig -multi_cor nr \
        -lth QR 1 -lth sig 0 -o $@/regulon$@.go.tab
-->
### Collecting the full set of promoters for the genome of interest

Before the proper analysis of the gene cluster, we will retrieve the promoter sequences of all the genes of the organism of interest, which will serve below to estimate the background model.

1.  Open a connection to the RSAT::Plants server (<http://plants.rsat.eu/>). On the left-side menu, select 'Sequence tools -\> retrieve sequence'.

2.  Choose 'Single organism -\> Zea\_mays.AGPv3.29' for the examples of this protocol (see **Note 1**). At the time of publication this corresponds to Ensembl Plants release 29, but that might change over time.

<!--JvH : this is a recall to a previous note, the number must thus be hard-coded but this can only be done at the end.  -->
1.  Choose 'Genes -\> all'; this will retrieve all upstream sequences of the maize genome.

2.  Set appropriate upstream bounds. Default values are -2000,-1. To replicate the work of Yu et al *(4)* these should be set to 'From' -1000 'To' +200.

3.  We recommend to tick the option 'Mask repeats', as plant genomes are frequently repeat-rich (see **Figure 1** and **Table 1**; maize genome contains 78% of REs). This option should not be used if you suspect the transcription factors of interest bind to repeated sequences.

4.  Press 'GO' and wait until the result page is displayed (see **Note **[4]). The last row of the results table should include an URL to the 'sequences' file, which must be saved. We will refer to this URL as '**all.fasta.URL**'. This FASTA-format file can also be stored as a local file on your computer, but note it can be rather large (52Mb in this example).
    <!-- http://floresta.eead.csic.es/rsat//tmp/apache/2016/02/06/retrieve-seq_2016-02-06.214631_BiEezu.fasta -->

### Analyzing upstream sequences of co-expressed genes

We will now retrieve the upstream sequences of a cluster of co-expressed genes, and use the tool *peak-motifs* to discover exceptional motifs in their promoters. The tool *peak-motifs* was initially conceived to discover motifs in ChIP-seq peaks, but it can also be used to analyze other sequence types, as illustrated here.

1.  Choose a cluster from **Table 2**, copy the corresponding gene IDs (last column) and paste them in a new text file, that you will store on your computer. Insert newline characters between genes[5].

2.  In the left menu of the RSAT server, click on 'retrieve sequence' to get a fresh form. Make sure that the option 'Genes -\> selection' is activated. Tick 'Mask repeats', and set the same size limits as for the whole collection of promoters: from -1000 to +200. Paste the list of IDs of your gene cluster (one gene ID per row).

3.  Press 'GO' and wait a few seconds until the result page is displayed. Inspection of these sequences might reveal N-masked sequence stretches, which correspond to annotated repeats. Save both 'query genes' and 'sequences' files to local files on your computer, we will refer to them as '**cluster.genes**' and '**cluster.fasta**' later on this protocol.

4.  Press the 'peak-motifs' button. **Peak sequences** are pre-filled with the cluster sequences selected on steps 7-8.

5.  Add a title for this job, such as 'E2F cluster'.

6.  Paste the '**all.fasta.URL**' on the 'URL of a sequence file available on a Web server' entry form on the right side, as **Control sequences**.

7.  Click on 'Reduce peak sequences' and set an appropriate value to **Cut peak sequences +/-**. Suppress the default value (500) and leave the field blank, to avoid having the sequences clipped to 1Kb (by default, peak-motifs clips the peaks to 500bp on each side of the peak centers).

8.  Click on 'Motif discovery parameters'. Select two algorithms: 'Discover over-represented words' (**oligo-analysis**) and 'Discover over-represented spaced word pairs' (**dyad-analysis**). Uncheck the program **position-analysis** (see **Note **[6]). Set 'Origin' to 'end' and 'Offset' to -200 (see **Note **[7]).

9.  Click on 'Compare discovered motifs with databases' and select appropriate databases which will be used to annotate any found motifs. For plant promoters, we recommend to check '*footprintDB-plants*', but you can also check other databases such as '*Athamap*', '*ArabidopsisPBM*' and '*JASPAR plants*' (see **Note **[8]). <!-- However, as annotations in other organisms are deeper, considerer adding generic repositories such as footprintDB or JASPAR to improve your annotations. 
    --> <!-- Bruno: I (JvH) think that it is a bad idea to compare plant motifs to those from other organisms, because it would be misleading for users who might think that they can expect conservation of the factors and their binding  across different Reigns of the tree of live. Do you have a good reason to do this ?  --> You can also upload your own collection of DNA motifs in TRANSFAC format.

10. Select **email** output and type your delivery address as these are costly jobs and your connection to the server might get interrupted otherwise. Press 'GO'.

11. After few seconds the server should have uploaded the sequences and display a page with the URL of the future result page. You can already click on this link: the result page will be periodically updated to show the progress of the analysis. At the end of the processing, a box will appear at the top of the result page, with a short summary of the discovered motifs, and links to different sections of the results. <!--http://localhost/rsat/tmp/_www/2016/02/07/peak-motifs.2016-02-07.014221_2016-02-07.014221_ko1Omo/peak-motifs_synthesis.html--> Once the job is complete click on the link **[Download all results (peak-motifs\_archive.zip)]** to **save the results** on your computer. You will later be able to uncompress this archive in order to check the result after its removal from the server (results are only available on the server for 72 hours after job completion). We also recomment to download the full set of discovered motifs, by clicking on the link **[Download all matrices (transfac format)]** and saving a local file named '**cluster.motifs.tf**'. This file contains all motifs in the form of position-weight matrices (PWMs) in TRANSFAC format.

On the result page, the section entitled *'Discovered motifs (with motif comparison)'* lists the discovered motifs, displays their sequence logos and their distribution along clustered sequences, in addition to top matches with the motif databases selected on **step 15**. The top motifs found by *oligo-analysis* and *dyad-analysis* are reported on **Table 3**.

### Negative control: random groups of genes

In this section, we propose a procedure to obtain an empirical estimation of the rate of false positives, by discovering motifs in the promoters of genes picked up at random.

1.  Select 'Build control sets -\> random gene selection'.

2.  Choose 'Organism -\> Zea\_mays.AGPv3.29' for the examples of this protocol.

3.  Set 'Number of genes' to the size of one the sample clusters on **Table 2**. For instance, the size of the negative control sets would be 18 for cluster E2F, 16 for cluster ABI4, and 56 for cluster WRI1. For convenience, in this tutorial only one random group is generated (the default), but this utility can generate several random groups in one go (see **Note **[9]).

4.  Press 'GO' and click the 'Next step' button 'retrieve sequences' at the bottom of the result page. In the retrieve-seq form, set the other parameters as above: from -1000 to +200, check the 'Mask repeats' option.

5.  Repeat steps 7-16. Save 'query genes' and 'sequences' files to local '**random.genes**' and '**random.fasta**' later on this protocol. The top motifs found by oligo-analysis and dyad-analysis on such a random cluster are reported on **Table 4**.

### Validating motifs by scanning promoter sequences

This part of the protocol is devoted to validating sequence motifs discovered by their over-representation, which are scanned against the original sequences from which they were discovered, plus, optionally, orthologous sequences from a related species (see **Note **[10]). The first goal of this section is to check whether the discovered motifs show patterns of occurrence along promoter sequences, and to see how many cluster sequences actually harbour them. This can be done empirically by comparing the results of expression-based motifs with those of shuffled motifs, which play the role of negative controls. A second goal is to investigate whether these regulatory motifs are conserved on orthologous promoters of a related plant, *Sorghum bicolor* in this case study.

<!-- I (JvH) don't understand the rationale of this section: it apparently contains two things: scanning sequences with a permuted matrix (negative control) and scanning sequences of orthologous genes. These two actions seem to be related to different purposes. -->
1.  Select 'Comparative genomics -\> get orthologs-compara'.

2.  Choose 'Reference organism -\> Sorghum bicolor' for the maize example.

3.  Upload file '**cluster.genes**' generated on **step 9**. Press 'GO' and finally press 'retrieve sequences' on the next screen.

4.  Repeat steps 4-6 and save 'the sequences' to local file '**cluster\_orths.fasta**'. <!-- 26. Repeat steps 22-25 with '**random.genes**', ultimately saving '**random_orths.fasta**'. -->

5.  Select 'Build control sets -\> permute-matrix'.

6.  Upload '**cluster.motifs.tf**' (obtained in **step 17**) and press 'GO'. Save the results file as '**cluster.motifs.perm1.tf**' (see **Note **[11]).

7.  Select 'Pattern matching -\> matrix scan (full options)'.

8.  On the sequence box paste the contents of '**cluster.fasta**' and, optionally, '**cluster\_orths.fasta**', if you wish to assess motif conservation. Alternatively, **steps 29-33** can be performed separately with maize and *S.bicolor* sequences.

9.  Upload file '**cluster.motifs.tf**' ans select 'transfac' format.

10. In the 'Background model' section select Markov order 2 and choose 'Organism-specific -\> Zea\_mays.AGPv3.29'. Press 'GO'.

11. Save the 'Scan result' file as '**cluster.scan.ft**' and press the 'feature map' button to draw a map of the matched motif instances.

12. Repeat steps 28-32 using the set of permuted PWMs '**cluster.motifs.perm1.tf**'.

### Interpretation of results

The last stage of the protocol is the interpretation of results, which requires having at hand results of both clusters of co-expressed genes and random clusters, which play the role of negative controls. **Figure 2** summarizes the results of clusters in **Table 2** compared to 50 random clusters of the same size. There are three types of evidence to look at, which will be discussed with the examples on this figure.

-   The **distributions of motif significance** yielded by *oligo-analysis* (A,E,I) and *dyad-analysis* (B,F,J). Motifs discovered in random clusters (grey bars) typically have significances below 4 <!-- Should we calculate the intervals? -->. The motifs found in ABI4 and WRI1 clusters (black bars) are not more significant than those of random gene sets of the same sizes, and should thus not be considered as reliable predictions. In contrast, the most significan oligomer found within E2F upstream sequences clearly supersedes those of random clusters, and a very similar motif is reported by *dyad-analysis*, with a lower but still strong significance. For this reasons, E2F motifs can be considered as promising predictions. <!-- Note by JvH: I suppress the word "confidence", which has a precise meaning in stats, because we do not actually compute confidence here"-->

-   The **distributions of scanning scores** (C,G,K) show to which extent motif matches in upstream sequences, including promoters of *S.bicolor* orthologues (dark boxes), depart from matches of permuted matrices (lighter boxes), used here as negative controls. In the example, the results for E2F motifs confirm their relevance, while the contrary is observed for ABI4. WRI1 results are somewhat in between.

-   The **distributions of scores in footprintDB** (D,H,L) describe how similar are the discovered motifs when compared to motifs (PWMs) annotated in footprintDB. Similarities are measured by the \(\text{maxNcor}\) score (see **Note **[12]). As to the random clusters, in each example 50 sets of motifs are compared to tootprintDB. The results indicate that a comparison to a motif repository, while essential for annotation purposes, is not particularly helpful in order to distinguish relevant expression-supported motifs from PWMs constructed in random sequence clusters. Indeed, in two out of three examples random clusters produce motifs that have matches in footprintDB with superior Ncor scores than the motifs experimentally confirmed by Yu et al *(4)*.

4. Notes
--------

4. References
-------------

1. MA Beer, S Tavazoie (2004) Predicting gene expression from sequence. Cell 117: 185–198.

2. TJ Sand O., van Helden J. (2008) Evaluating the prediction of cis-acting regulatory elements in genome sequences, In: Frishman, D. and Valencia, A. (eds.) Modern genome annotation, pp. 55–89 Springer.

3. J Koschmann, F Machens, M Becker, J Niemeyer, J Schulze, L Bulow, DJ Stahl, R Hehl (2012) Integration of bioinformatics and synthetic promoters leads to the discovery of novel elicitor-responsive cis-regulatory sequences in Arabidopsis. Plant Physiol. 160: 178–191.

4. CP Yu, SC Chen, YM Chang, WY Liu, HH Lin, JJ Lin, HJ Chen, YJ Lu, YH Wu, MY Lu, CH Lu, AC Shih, MS Ku, SH Shiu, SH Wu, WH Li (2015) Transcriptome dynamics of developing maize leaves and genomewide prediction of cis elements and their cognate transcription factors. Proc. Natl. Acad. Sci. U.S.A. 112: E2477–2486.

5. T Schmidt, J Heslop-Harrison (1998) Genomes, genes and junk: the large-scale organization of plant chromosomes. Trends in Plant Science 3: 195–199.

6. A Sebastian, B Contreras Moreira (2014) footprintDB: a database of transcription factors with annotated cis elements and binding interfaces. Bioinformatics 30: 258–265.

7. KL Howe, B Contreras-Moreira B, N De Silva, et al. (2019) Ensembl Genomes 2020-enabling non-vertebrate genomic research. Nucleic Acids Res. 48: D689–D695

5. Notes
--------

[1] As gene models can change from one assembly to another it is important to use the right assembly version, which is indicated for each genome on **Table 1**. If the assembly of interest it not available on RSAT::Plant server, please contact the first author.

[2] Smaller clusters (GBF6,GLK2,CIB1,GLK1,HOX16,RS2) were left out of the analysis, as the statistical approaches in this protocol require at least ~10-15 sequences. Cluster MYB59 was left out due to space restrictions but its results can be browsed at <http://plants.rsat.eu/data/chapter_expression_clusters>

[3] A crucial parameter to evaluate the results of motif discovery is to estimate the rate of false positives (FP). RSAT programs compute a significance score, which is the minus log of the expected number of false positives (\(\text{e-value} = 10^{-\text{sig}}\)). For example, a motif associated with a significance of 1 should be considered as poorly significant, since on average we would expect \(10^{-1} = 0.1\) false positives, i.e. one FP every 10 random trials. In contrast, a significance of e.g. 16 is very promising, since on average such a result would be expected every \(10^{-16}\) random trials. However, the theoretical significance relies on the correctness of the background model (computed here as k-mer and dyad frequencies in the whole set of promoters). In some cases, sets of plant promoters can discard from the theoretical model, due to heterogeneity of the input (e.g. inclusion of repetitive sequences). The negative control consists in measuring the significance obtained by submitting a random selection of promoters from the organism of interest (maize in the example). Although each of these genes is likely to be regulated by one or more transcription factors (and its promoter should contain corresponding binding sites), in principle the random set as a whole should not be co-regulated, so that the elements would differ from gene to gene, and there should thus be no over-represented motif in their promoters.

[4] Should the connection to the server interrupt it might be safer to go back and choose 'email' as delivery option.

[5] It is crucial to have one gene ID per row for submitting queries to retrieve-seq, because only the first word of each row is considered as a query.

[6] This program is generally relevant when analyzing sets containing a large number of sequences such as ChIP-seq peaks or genome-wide promoter sets.

[7] The option *'Origin'* indicates the reference position relative to each sequence (start, center or end). When this option is set to 'end', the coordinates are computed relative to the end of the sequence, with negative values indicating upstream location. The option *'Offset'* enables to shift the reference point by a given number. For the current example, setting the offset to -200 will give coordinates from -1000 to +200, the 0 corresponding to the TSS.

[8] Plant transcription databases are unfortunately still very fragmentary, so one might be tempted to check more complete collections such as *footprintDB* or *JASPAR core all*. However, the results should be interpreted with caution, because there is no conservation of cis-regulation between Plants and other Kingdoms of the tree of life.

[9] Clearly, more than one random cluster should be evaluated, as suggested on **Figure 2**, where the results of up to 50 random groups are displayed next to the clusters of *(4)*.

[10] Orthologues reported are annotated in Ensembl Compara, generated by a pipeline where maximum likelihood phylogenetic gene trees play a central role. These gene trees, reconciled with their species tree, have their internal nodes annotated to distinguish duplication or speciation events, and thus support the annotation of orthologous and paralogous genes, which can be part of complex one-to-many and many-to-many relations. Adapted from: <http://www.ensembl.org/info/genome/compara/homology_method.html>.

[11] This will permute the columns of the input PWMs producing matrices with the same information content but different consensus.

[12] 'Ncor' is the relative width-normalized Pearson correlation when aligning PWMs with *matrix-scan*. This normalized score prevents spurious matches that would cover only a subset of the aligned matrices (e.g. matches between the last column of the query matrix and the first column of the reference matrix, or matches of a very small motif against a large one).
