https://liambai.com/protein-evolution/
Proteins: molecular machines that orchestrate almost all activity in our biological world.
Amino acids make up proteins and specify their structure and function - Anfinsen's Dogma (or the Thermodynamic Hypothesis)

MSAs (Multiple sequence Alignmnet): The co-evolutionary coupling data provides the **strong physical constraints** needed to *navigate complex structural landscapes and achieve atomic-level accuracy*
- **When can I group the proteins in same MSA [group homologous protein, variants of a protein]?*** share sequence similarity because they came from a common ancestor — function is usually similar but can drift, especially between paralogs.
    - They should be from common ancestor (homologous proteins descended from the same ancestral gene.) -shared ancestory.
    - Orthologs (same gene, different species) typically do retain the same function.
    - Paralogs (duplicated genes in the same species) start from a common ancestor too, but often diverge in function over time.

- If an amino acid at position A mutates to a larger residue, it might destabilize the local 3D structure unless position B (which touches it) mutates to a smaller residue to accommodate the change.The Signal: When you look across a Multiple Sequence Alignment (MSA) of homologous proteins, positions A and B will appear to mutate in tandem (co-vary) across different species. So the core idea behind MSA is = **amino acid positions that tend to co-vary in the MSA tend to interact with each other in the folded structure**, often via direct 3D contact, which means **Evolution helps identify which mutations co-exists** [Find out co-evolve sights in MSA to ultimately predict protein 3D structure.]


**Multiple Sequence Alignment Models::::**
<<search of a distribution:>>
The basic idea here is to learn what makes a sequence look like a valid/functional member of a protein family, Learn a model P(A) using given sequence A = (A1, A2, A3.....An) that belong to same/one protein family.

[a.] Position-specific scoring matrix [PSSM] - To model evolutionarily conserved position.
    This gives the energy-like score:
        E(A)= ∑i [fi(Ai)]
    The frequencies of observing each amino acid at each position. 
    
    E(A) is big when the amino acid frequencies in each position of A matches the frequency patterns observed the in MSA – and small otherwise.

[b.] 
    Modelling co-variation between pairs of positions instead of single position.
    fij(Ai, Aj) be the frequency of observing amino acid Ai at position  i and amino acid Aj at position j.

​       $$E(A) = \sum_{1 \le i \le j \le L} J_{ij}(A_i, A_j) + \sum_{1 \le i \le L} f_i(A_i)$$ 

    They call this the Potts model [a mathematical model that is used to describe a system of interacting particles, which could be things like atoms, molecules, or even pixels] , and a fancy name for the **energy function is the Hamiltonian**

    
    Till now modelling Jij(Ai, Aj) - It’s a local m seasurement. Imagine a case where positions i and  j each independently interact with position  k, though they do not directly interact with each other. With this transitive correlation between i and j, the nearsighted fij would likely overestimate the interaction between them meaning the interaction between i,j will be captured although it doesn't exists due to this transitive property.

    To disentangle such direct and indirect correlations, we want a global measurement that accounts for all pair correlations.

    Direct Coupling Analysis (DCA) -



https://tianyu-lu.github.io/communication/protein/ml/2021/04/09/Potts-Model-Visualized.html
> A Potts model is a probabilistic model.
> For protein sequences, it is usually used to model a homologous set of sequences, i.e. sequences which likely evolved from one ancestral protein sequence.
> But what do we mean by “model”? A Potts model assigns each sequence a number which represents how likely that sequence belongs to this homologous set.

**Evolution of protein sequences work?**
- The **structural proximity of two amino acids can impose constraints on what mutations are allowable**.
- For example, certain amino acids are not allowed in the case of charged amino acids. This implies that by finding pairs of amino acids which **evolve together (coevolve)**, rather than evolving independently, we can infer that these **two positions are likely to be structurally adjacent**
    > Somehow the information of coevolving residues need to be captured in the Potts model. A Potts model does this by capturing the sitewise and pairwise frequencies in the sequence data. Sitewise means for some position i along the sequence, what are the probabilities of all the possible amino acids at that position. Pairwise means for some pair of positions i and j, what are the probabilities of all 21 by 21 pairs of amino acids in those two positions. 
    > It’s able to do this owing to an important concept in machine learning: inductive bias (assumptions that the model makes eg: CNN assume that pixels next/in close proximity are related and ones located far or to the very end are irrelated).


    The J parameter is a (L by 21) by (L by 21) matrix that looks like this - The original matrix is L X L where L are number of position in the sequence and each block in LXL matrix is 21X21 [which is 20 amino acid + one blank (-)].Each value in LXL - summarize each 21×21 block which is one position-pair coupling score which is maximum.

    It also includes a parameter h which models the sitewise frequencies of sequences, which is LX21.

    Notably, the J parameter captures pairwise correlations. However, just looking at the correlations between amino acid pairs is not enough. If A and B coevolve, but B and C also coevolve, this means A and C coevolve as well but they are not necessarily structurally adjacent.

   **To disentangle such direct and indirect correlations, we want a global measurement that accounts for all pair correlations.**

    **Direct Couplings Analysis:**
   **[a.] Mean-field DCA (the simplest version):**
    Step 1 — One-hot encode the MSA
        Given an MSA with N sequences, each of length L, over a 21-symbol alphabet (20 amino acids + gap):

        For each sequence, at each position i, the observed amino acid is one of 21 categories. One-hot encoding turns this into a 21-dimensional indicator vector.

        Example: alphabet order (-, A, C, D, E, ..., Y). If position i is A:
        x_i = (0, 1, 0, 0, 0, ..., 0)     # 1 in the "A" slot, 0 elsewhere
        
        **Stacking N such sequences gives an N × (LX21) binary data matrix**
    
    Step 2 — Single-site frequencies → h
        For position i, amino acid a:
            f_i(a) = (1/N) * Σ_n  x_i^(n)(a)
        This is just the column mean of the one-hot data — **the fraction of sequences with amino acid a at position i**. Collecting these gives the full table is 21×L (equivalently L×21, just transposed matches the shape of h from earlier).
    
    Step 3 — Pairwise frequencies
        For positions i, j and amino acids a, b: — this is a normalized joint co-occurrence count: what fraction of sequences have a at i and b at j at the same time.
            f_ij(a,b) = (1/N) * Σ_n  x_i^(n)(a) · x_j^(n)(b)
       `    
        fixed pair of positions (i, j), f_ij(a,b) is a 21×21 matrix — one entry for every combination of amino acid a (at position i) and amino acid b (at position j)

                f_ij(a,b) =
                            b=(-)   b=A    b=C   ...   b=Y
                    a=(-)  [  f(-,-)  f(-,A) f(-,C) ... f(-,Y) ]
                    a=A    [  f(A,-)  f(A,A) f(A,C) ... f(A,Y) ]
                    a=C    [  f(C,-)  f(C,A) f(C,C) ... f(C,Y) ]
                    .     [    ...     ...    ...   ...  ...  ]
                    a=Y    [  f(Y,-)  f(Y,A) f(Y,C) ... f(Y,Y) ]

        Each entry is the fraction of sequences in the MSA that have amino acid a at position i and amino acid b at position j, simultaneously.

    Step 4 — Covariance matrix blocks
    Using the standard identity Cov(X,Y) = E[XY] − E[X]E[Y]:
        C_ij(a,b) = f_ij(a,b) − f_i(a) · f_j(b)

    Off-diagonal blocks (i ≠ j): the ones that matter for coevolution — how amino acid a at position i covaries with amino acid b at position j.
        
    Step 5 — Assemble the full covariance matrix
    Stack every 21×21 block C_ij for all i, j = 1...L into one matrix of size (L·21) × (L·21):
                    j=1 block      j=2 block     ...    j=L block
        i=1  [ C_11(21×21)   C_12(21×21)   ...   C_1L(21×21) ]
        i=2  [ C_21(21×21)   C_22(21×21)   ...   C_2L(21×21) ]
        .   [     ...            ...       ...       ...    ]
        i=L  [ C_L1(21×21)   C_L2(21×21)   ...   C_LL(21×21) ]xs

    Same L×L grid-of-blocks structure as J, but populated with raw covariances rather than couplings. This matrix is symmetric and positive semi-definit

    Step 6 — Regularize and invert
    C_reg = C + λI
   
   Then invert: J ≈ −(C_reg)^(-1)
   This inversion is the step that separates direct from indirect coupling. It's mathematically identical to computing a precision matrix (inverse covariance) in a Gaussian graphical model — the same operation used to get partial correlations.








https://cs.rice.edu/~ogilvie/comp571/pssm/
Global and local alignment tools like BLAST are typically used to search for and align homologous sequences which are related by common descent. However we may also wish to search for short sequence motifs which are similar primarily because they have a common function.

These sequences might be homologous if they are related through common descent, but they may also have evolved convergently (need to perform similar functions due to similar environmental challenges).
> Certainly extremely short and simple functional motifs like splice sites can evolve independently again and again, and nobody would claim that every intron1 in a genome is homologous.
> but when looking for short motifs we are usually interested in them for their common function rather than their common origin.
> One way to find these motifs is to use a position-specific score matrix, or PSSM.  - a PSSM specifies the scores for observing particular amino acids or nucleotides at specific positions.


?????? AlphaFold does not use external DCA tools like EVcouplings. Instead, AlphaFold generates raw MSAs using search tools like HHblits and Jackhmmer.
???????



https://www.youtube.com/watch?v=uPoFdCUqBWk
**Protein Language models:**
https://liambai.com/protein-evolution/






[a.] Sequence Alignment tools
[b.] Coupling Modelling