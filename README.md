# lisp-seq-cause-search

**Problem**

Identifying causes of rare events remains a challenge across disciplines such as natural language processing, software safety, and bioengineering. Causal information can often be contained in sequences of discrete tokens (e.g. words, computation states, nucleotides), associated with a discrete or continuous value indicating an occurrence or property of a rare event. 

The presented algorithm enables enrichment of sequences associated with rare events by searching for concise logical representations. The algorithm i) is scalable in the number of considered sequence positions and ii) provides an interpretable search space for identification of potential causal relationships.

**Algorithm**

An input data set consists of sequences of discrete tokens. Each sequence is associated with a continuous or discrete value.

A rule is a conjunction of disjunctions. A conjunction across sequence positions constrains a rule, whereas XOR disjunctions across literals at each sequence position relax a rule. A rule satisfies a subset of sequences in the input data set. An objective function maps the values of satisfied sequences to a real value. 

The search algorithm provides a local search for rules with a minimized or maximized value of the objective function, and a minimized sequence variance for interpretability purposes. The latter property is formulated outside the objective function and is enabled by i) constructing each disjunction in a start rule from a set of literals, such that each literal is expected to be satisfied in at least one optimal rule, and ii) setting a preference for literal deletion over literal addition during a search step.

**Application - Bioengineering, Proteomics**

Protein-protein interactions provide an effective way to modulate protein targets for therapeutic purposes. However, binding events in protein-protein interaction data are generally rare and can be associated with a high sequence variance across many interdependent amino acid positions. As a consequence, it remains challenging to mechanistically link specific amino acid changes to observed affinity and selectivity changes. 

**Data**

The data set published by Stiffler et al. (2007) was used as a source of verified interaction pairs between PDZ domains and peptide partners. The entire data set, comprising 22134 interaction pairs, was acquired under controlled standardized conditions and was not pooled from different sources.

659 binding interactions and 21475 non-binding interactions were identified with protein microarrays. All identified binding interactions and a subset of non-binding interactions were verified with fluorescence polarization. Although the total number of binding interactions was relatively small, almost all position-amino-acid pairs were involved in binding interactions (green). A small subset of position-amino-acid pairs were involved in only non-binding interactions (orange).

![binding_s](https://github.com/alfin3/lisp-seq-cause-search/blob/master/images/data.png)

**Results**

16 sequence positions across PDZ domains and 5 sequence positions across peptides were considered. A rule was defined as a conjunction of 21 XOR disjunctions of amino acid literals. Given a rule, the value of the objective function was defined as the ratio of binding data instances to non-binding data instances satisfied by the rule. 

The size of the search space was given by

![form](https://github.com/alfin3/lisp-seq-cause-search/blob/master/images/space_size.gif), 

where ![nj](https://github.com/alfin3/lisp-seq-cause-search/blob/master/images/nj.gif) was the number of literals at the position ![j](https://github.com/alfin3/lisp-seq-cause-search/blob/master/images/j.gif). A mixture of hill-climbing and random walk was used. Each search was started with the rule that included all literals that occured in binding data instances. Literals were preferably deleted from and sometimes added to a rule during a search. After each deletion of a literal, the resulting rule retained the literals of the preceding rule that did not change its set of satisfied instances ("non-essential" literals). This feature improved search results, demonstrating that the choice of a better representation of search solutions could improve the performance of a local search algorithm. "Non-essential" literals were deleted in a final output rule. 

![combo](https://github.com/alfin3/lisp-seq-cause-search/blob/master/images/search_runs.png)

Output rules were mapped to PDZ-peptide interaction matrices (see two example below). Each interaction pair in a matrix contained a measured Kd value (green) representing a binding interaction, or -1.0 or -100.0 representing a non-binding interaction where no Kd value could be measured. 

*Rule 1*
![rule](https://github.com/alfin3/lisp-seq-cause-search/blob/master/images/rule1.png)

1) selectivity transition >SHANK1_1/1_216X_P_1Q3PA vs. >SHANK3_1/1_217X_P_1Q3PA:
the first 5 peptides ended with TRL and the single amino acid change from K to R blocked only the peptide starting with AQ but not the other four peptides that continued to bind strongly.

2) affinity transition >SHANK3_1/1_217X_P_1Q3PA vs. >NHERF-1_2/2_194X_P_2OZFA:
all 6 peptides were binding to each PDZ, but Kd values for the peptides ending with TRL increased when binding to >NHERF-1_2/2_194X_P_2OZFA.

*Rule 2*
![rule2](https://github.com/alfin3/lisp-seq-cause-search/blob/master/images/rule2.png)

1) S A in >PSD95_1/3_28X_P_1KEFA and >SAP97_1/3_29X_P_1ZOKA did not change the binding profile from previous interactions

2) but additional changes SRA D in >SCRB1_2/4_37X_P_1WHAA left only IETHV interacting,

3) but then again additional changes N V K Q in >OMP25_1/1_8X_P_2ENOA restored the initial interaction profile.

The starter function in stochastic-search.lisp can be used to run a search (e.g. (starter "DATA_full.txt") with a pre-seeded random state). 