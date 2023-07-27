# Empirical Models and Signalling Fractions of Garden-Path Sentences

## Description of the method
Our aim is to collect the probability distributions over the possible parses of different fragments of a garden-path sentence. We then proceed as follows:
1. For every sentence fragment, we use the transformer-based language model BERT to obtain predictions of the completions of the sentence, by masking all of the words in the sentence not within the fragment. For example, given the sentence:
    >*The faithful employees understood the technical contract would be changed very soon.*

    and the fragment:
    >*The faithful employees understood the contract*

    we would collect the BERT scores for the input sequence:
    ```
    The faithful employees understood the contract [MASK] [MASK] [MASK] [MASK] [MASK]
    ```
    The (logit) scores $s$ are then converted into probabilities $p$ as follows:
    $$p=\frac{1}{1+e^s}$$

    We also decided to exclude the non-textual completions, and the probabilties were subsequently re-normalised.

2. Each of the predictions are then parsed using the dependency parser spaCy.

3. The obtained parses are then restricted to the fragments of interest by forgeting about the dependecies of the previously masked words.

4. The BERT probabilities of the completions leading to the same partial parses are then added together to give the final probability of the partial parse for a given fragments.

The sentence fragments will constitute our set of **contexts**, and the associated partial parses the set of **outcomes**. In addition, in order to model the reading process, we only consider fragments of increasing lengths, e.g.:
- The
- The faithful
- The faithful employees
- etc.

We also work with sentences coming from the dataset of [1] (subsequently refered to as the Sturt et al. dataset) and [2] (subsequently refered to as the Grodner et al. dataset).

The Sturt et al. dataset consist on:
- 32 NP/S and 32 NP/Z gardent-path sentences and disambiguated versions of each the garden-path sentences. Each sentence is split into 4 different regions
- Region-by-region self-paced reading times (averaged across all of the sentences)

The Grodner et al. dataset consists on:
- 20 base NP/S and 20 base NP/Z sentences

such that:

- Each of the base sentence has a modified version, where a descriptive complement is added to the subject of the main verb
- Each of the garden-path sentence (modified and unmodified alike) also have disambiguated versions
- Each sentence is split into regions
- We have the average word-by-word self-paced reading time for each region of each of the different sentence types (averaged across sentences).

## How to read the dataset
There are two files in the repository:
- `stimuli.csv`
- `dataset.json`

The `*.csv` file contains the set of all of the sentences of the two datasets that we have used for analysis; the `*.json` files contains the empirical models and associated $SF$. 

### The `stimuli.csv` file
The stimuli file contains 6 columns:
1. `Dataset`: this corresponds to the dataset from which the sentence is from, namely `Grodner` or `Sturt`;
2. `Index`: this is a way of giving a identifier for the sentence. This helps finding the equivalent ambiguous/unambiguous or modified/unmodified versions of the same sentence;
3. `Ambiguity`: this shows whether the sentence is ambiguous (labelled `A`) or unambiguous (labelled `U`);
4. `Modified`: this should be understood as a Boolean stating whether the sentence is modified (labelled $1=True$) or unmodified (labelled $0=False$). Also note that the distinction does not apply for the Sturt et al. dataset, the modified labels are set to $-1$ for the sentences coming from this dataset;
5. `Stimulus`: these are the actual sentences. They are also split into the regions from the respective articles (the separator is `/`).

### The `dataset.json` file
The dataset file is a dictionary of the general form:
```
spacy-model
    bert-model
        "model": empirical-model
        "SF": signalling-fractions
```
The `spacy-model` correspond to the spaCy version used for parsing, e.g. `sm` correspond to the `en_core_web_sm` spaCy model. Similarly, the `bert-model` corresponds to the BERT model used for predictions, e.g. `BERT-base` corresponds to `bert-base-cased`.

The empirical models are structured as follows:
- The keys correspond to the BERT inputs, therefore, the contexts can be extracted from it by removing all of the `[MASK]`. 
- The values are themselves dictionaries. In these dictionnaries:
    - The keys are representations of the (partial) dependency parses. For each word of the context, their *head* is encoded as the index (starting from $0$) of the corresponding word in the sentence; the dependency type is ommitted. The set of heads is then concatenated with separator `_`.
    - The values are the associated probabilities.

The signalling fractions quoted correspond to sequence of signalling fractions that can be obtained by considering empirical models of two consecutive contexts. They should quantify the difficulty of parsing a given word; e.g. the 1st $SF$ for the sentence *The faithful employees understood the technical contract would be changed very soon* would quantify the difficulty of parsing the word *faithful*, 2nd $SF$ would correspond to the difficulty of parsing the word *employees* etc.

## References
[1] Sturt, Patrick, Martin J. Pickering, and Matthew W. Crocker. "Structural change and reanalysis difficulty in language comprehension." *Journal of Memory and Language* 40.1 (1999): 136-150.

[2] Grodner, Daniel, et al. "Against repair-based reanalysis in sentence comprehension." *Journal of Psycholinguistic Research* 32 (2003): 141-166.