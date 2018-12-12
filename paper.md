# 1. Hi-C
&nbsp;1. [Introduction](#11)  
&nbsp;2. [Protocol](#12)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.1. [Cross-linking](#121)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.2. [Digestion](#122)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.3. [Selection](#123)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2.4. [Sequencing](#123)  
&nbsp;3. [Analysis](#13)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.1. [Contact matrix](#131)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.1.1. [Concept](#1311)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.1.2. [Creation](#1312)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.2. [TAD Calling](#132)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.2.1 [Intuition](#1321)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.2.2 [Algorithm](#1322)  

## 1.1. Introduction<a name="11"></a>

Hi-C is a chromosome conformation capture (3C) technique used to quantify interactions between different sections of the genome. Unlike older 3C techniques, Hi-C discovers all genomic interactions, as opposed to only interactions between one locus and the rest of the genome, for example.

One application of the Hi-C technique is to identify Topologically Associated Domains (TADs) within the chromosome. TADs are regions of the chromosome that interact with themselves more frequently than they interact with other regions of the chromosome. They are a general property of genomes, and contribute to proper gene regulation and other nuclear functions. TADs typically consist of 105 to 106 contiguous base pairs, and are separated from each other by boundaries which prevent interaction between TADs. Since Hi-C quantifies chromosome interactions, it is a very useful tool for identifying these highly self-interacting regions.

## 1.2. Protocol<a name="12"></a>



### 1.2.1. Cross-linking<a name="121"></a>

Cross-linking is a process used to create covalent links between interacting sections of chromatin. Chromatin interactions are mediated by DNA-binding proteins, and treating the cell with formaldehyde creates covalent bonds between chromatin and DNA-binding proteins, and between DNA-binding proteins and other proteins.

### 1.2.2. Digestion<a name="122"></a>

Digestion of the genome is a process which is used to produce short fragments of cross-linked chromatin. After cross-linking, the cell is lysed and homogenized. Then DNA is digested into fragments through the use of the HindIII restriction enzyme, which leaves 5’ sticky ends on each fragment of DNA. This process results in relatively short interacting chromatin fragments that are cross-linked together via DNA-binding proteins.

### 1.2.3. Selection<a name="123"></a>

In order to later isolate the fragments of interacting chromatin from the rest of the cell, the ends of each fragment of DNA are filled in with nucleotides, including a biotinylated nucleotide. The blunt ends of the DNA are then ligated under extremely dilute conditions which favor ligation between physically close, cross-linked fragments. The digested chromatin mixture is then purified through the use of proteinase and RNase. Exonucleases are used to remove biotinylated nucleotides from unligated (ie. unlinked) DNA fragments. Fragments still containing biotin are then pulled down and purified using streptavidin beads. 

### 1.2.4. Sequencing<a name="124"></a>

After the DNA is purified, it must be sequenced and analyzed in order to identify interacting sections of the genome. First, the purified DNA is sequenced using Illumina paired end sequencing. Then, each end of the read is aligned to the genome independently, revealing a pair of genome loci which interact with each other.

## 1.3. Analysis<a name="13"></a>
### 1.3.1. Contact matrix<a name="131"></a> 

#### Concept<a name="1311"></a>

A contact matrix is a heat map used to visualize interactions between different sections of the chromosome (or between different sections of the whole genome). The x- and y-axes represent loci on the chromosome in linear order. A dark colored square in the contact matrix represents a pair of loci which have many interactions between them, and a light colored square represents a pair of loci which have few interactions between them.

#### Creation<a name="1312"></a>

In order to create a contact matrix, the chromosome (or genome) must be divided into non-overlapping bins of equal size. A bin is a section of contiguous DNA for which Hi-C data will be grouped together. The bins may contain different numbers of base pairs based on the resolution necessary for the desired analysis.

Once bins have been established, the number of interactions between each pair of bins must be counted. These counts are then plotted onto the graph as a heat map, which higher numbers of interactions represented as darker colors, and lower numbers of interactions represented as lighter colors.

### 1.3.2. TAD Calling<a name="132"></a>
#### Intuition<a name="1321"></a>

Using the contact matrix, the boundaries between TADs can be identified (ie. the TADs can be called). Since TADs are contiguous regions of the DNA which interact most strongly with themselves, upstream sections of the TAD will have more downstream contacts, and downstream sections of the TAD will have more upstream contacts. A TAD boundary can be called at the point along the chromosome where bins switch from having mostly upstream contacts to mostly downstream contacts.

The number of upstream or downstream contacts a bin has can be quantified using the Directionality Index (DI). The DI is calculated as follows:

In the above equation, note that the second factor the the DI can never be negative, so the sign of the DI depends on the first factor. If the bin has more upstream contacts than downstream contacts (ie. A > B), then the DI will be positive. If the bin has more downstream contacts than upstream contacts, then the DI will be negative.

#### Algorithm<a name="1322"></a>
    
In order to model the TAD calling problem, a hidden Markov model is used. A hidden Markov model is a consists hidden states which transition into other hidden states, and which give rise to observable outputs. The probability of a hidden state transitioning into another hidden state is known as the transition probability. The probability of a hidden state producing a certain observable output is known as the emission probability.

In the TAD calling model, the observable outputs represent the DIs observed from the contact matrix. The hidden states represent the true DIs underlying the experimental observations, which are unknown. In order to infer the true DIs, the transition and emission probabilities must first be discovered using an expectation maximization algorithm. Then, the true DIs can be inferred using the forward-backward algorithm.

Once the true DIs have been inferred, TAD boundaries can be called at loci where the value of the true DI shifts from negative to positive.
