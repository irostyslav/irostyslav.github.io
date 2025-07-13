---
title: "Search Engine in Rust"
# author: "Francesco Tomaselli"
# authorAvatarPath: "/avatar.jpeg"
date: "2024-02-01"
summary: "A search engine overview and Rust implementation, supporting free and boolean ranked queries, efficient disk and memory usage, and spelling correction."
description: "A search engine overview and Rust implementation, covering text pre-processing, indexing, query resolution, and index compression."
toc: true
readTime: true
autonumber: true
math: true
tags: ["information-retrieval", "rust"]
showTags: false
---

I have always been fascinated by search engines and their capabilities: 
finding relevant documents in a pool of millions is certainly an incredible task, 
so I decided to dive deep into this topic. This was the perfect way to start with Rust. 

All the code is available on my [Github](https://github.com/tomfran/search-rs)
profile, feel free to have a look. 
This article is also published on [Medium](https://medium.com/@tomfran/building-a-search-engine-in-rust-c945b6e638f8).

## What is an Inverted-Index

The foundation of a search engine is an inverted index. 
The idea is to have a dictionary of terms, usually called **vocabulary**, 
and for each word a list of documents where it appears. This list can contain 
additional information, such as document frequency or positions. 
Those elements are usually called **postings**, hence those lists are **postings lists**.

### Example
Starting from two documents:

1. I did enact Julius Caesar: I was killed i' the Capitol; Brutus killed me.
2. So let it be with Caesar. The noble Brutus hath told you Caesar was ambitious.

We obtain an index as such, for the sake of the example only doc ids are shown in the postings lists:

$$
\begin{align*}
\text{ambitious} & \longrightarrow [2] \\\\
\text{be} & \longrightarrow [2] \\\\
\text{brutus} & \longrightarrow [1, 2] \\\\
\text{caesar} & \longrightarrow [1, 2] \\\\
\text{capitol} & \longrightarrow [1] \\\\
\text{did} & \longrightarrow [1] \\\\
\text{enact} & \longrightarrow [1] \\\\
\text{hath} & \longrightarrow [2] \\\\
\text{i'} & \longrightarrow [1] \\\\
\text{it} & \longrightarrow [2] \\\\
\text{julius} & \longrightarrow [1] \\\\
\text{killed} & \longrightarrow [1] \\\\
\text{let} & \longrightarrow [2] \\\\
\text{me} & \longrightarrow [1] \\\\
\text{noble} & \longrightarrow [2] \\\\
\text{so} & \longrightarrow [2] \\\\
\text{the} & \longrightarrow [1, 2] \\\\
\text{told} & \longrightarrow [2] \\\\
\text{was} & \longrightarrow [1, 2] \\\\
\text{with} & \longrightarrow [2] \\\\
\text{you} & \longrightarrow [2]
\end{align*}
$$

### Extracting words from documents

In the above example, we divided the documents into words by first removing all punctuation, lowering the text, and finally splitting words in whitespace. This is an example of **tokenization**.

There exist various techniques and one can be far more sophisticated with this task, for instance, whitespace splitting could lead to problems with multi-token words, such as *San Francisco*. For the sake of the project, we apply this simple technique nonetheless.

After tokenization, one could normalize the tokens. Terms such as *house* and *houses* should be counted as one key in the vocabulary, arguably also *be* and *was* could be accumulated.

A simple approach for this task is stemming, the idea is to reduce each word to its base form, for instance, by dropping a final *s*. A well-known algorithm and the one in use in the project is the [Porter Stemmer](https://tartarus.org/martin/PorterStemmer/).

Another way of doing this is **lemmatization**, which refers to properly using a vocabulary with morphological analysis.

Here is an example of the two techniques combined: 

$$\text{So many books, so little time.}$$
$$\downarrow$$
$$\text{so}, \text{mani}, \text{book}, \text{so}, \text{littl}, \text{time}$$

And here is the code example: 

```rust
// build regex r"[^a-zA-Z0-9\s]+"
// and Porter Stemmer

pub fn tokenize_and_stem(&self, text: &str) -> Vec<String> {
    self.regex
        .replace_all(text, " ")
        .split_whitespace()
        .map(str::to_lowercase)
        .map(|t| self.stemmer.stem(&t).to_string())
        .collect()
}
```

## Answering queries

The most important feature of a search engine is responding to queries. Websites such as Google made popular free-text queries, where you input a phrase and get documents ranked based on relevance. 

There exist also boolean queries, you might want documents containing both terms hello and world, but not man. This type is certainly more limited than the free ones, but they can be quite useful.

### Query pre-processing

One key aspect of reliably answering queries is pre-processing. We want to treat user inputs as if they were documents.

Keeping the stemmer example in mind, if we searched for the query little books without stemming it, we would not find anything, as the term *little* becomes *littl*, and *books* is transformed to *book*.

It is therefore really important to maintain **consistency** in documents and query **normalization**.


### Boolean Queries

We have built an index where for each term we quickly have documents containing it. Executing a boolean query is nothing more than sorted lists intersections, unions, and negations.

For instance, given the previous toy index, we can search for 
documents containing both the words *let* and *was* as such: 

$$
\begin{align*}
\text{let} \land \text{was} &= \text{intersect}([1], [1, 2])\\\\
&= [1]
\end{align*}
$$

Similarly, or operation becomes list merge: 

$$
\begin{align*}
\text{let} \lor \text{was} &= \text{merge}([1], [1, 2])\\\\
&= [1, 2]
\end{align*}
$$

Finally, not builds an inverse of the list: 

$$
\begin{align*}
\lnot \\\; \text{let} &= \text{inverse}([1])\\\\
&= [2]
\end{align*}
$$

To make a boolean expression easily parsable, we can transform it in its postfix notation using the 
[Shunting yard algorithm](https://en.wikipedia.org/wiki/Shunting_yard_algorithm), 
and then use a stack to execute it, here is an example:

$$
\begin{align*}
\text{original} &= \text{let} \land \text{was} \lor \lnot \\\; \text{me} \\\\\\
\text{postfix} &= \text{let} \\\; \text{was} \land \text{me} \\\; \lnot \\\; \lor
\end{align*}
$$

### Free-text queries

While boolean queries are certainly powerful, we are used to interacting with search engines via free-text interrogations. Also, we prefer to have results sorted by relevance, instead of receiving the ordered by id as in previous cases.

Given a free query, we first tokenize and stem it, and then, for each term, retrieve all documents, just like a boolean or query.

**BM25 score**

To obtain the final scoring function, we start with estimating term relevance in each document, 
the function in use here is [BM25](https://en.wikipedia.org/wiki/Okapi_BM25): 

$$\text{BM25}(D, Q) = \sum_{i = 1}^{n} \\\; \text{IDF}(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot \Big (1 - b + b \cdot \frac{|D|}{\text{avgdl}} \Big )}$$

Where the inverse document frequency is computed as: 

$$\text{IDF}(q_i) = \ln \Bigg ( \frac{N - n(q_i) + 0.5}{n(q_i) + 0.5} + 1 \Bigg )$$

The terms are: 
- $f(q_i, D)$: number of times query term $i$ occurs in document $D$;
- $|D|$: length of the document $D$ in words;
- $\text{avgdl}$: average length of the documents in the collection;
- $k_1$ and $b$ are hyperparameters, we used, $1.2$ and $0.75$ respectively.

Here is the code example

```rust
let mut scores: HashMap<u32, DocumentScore> = HashMap::new();

let n = self.documents.get_num_documents() as f64;
let avgdl = self.documents.get_avg_doc_len();

for (id, token) in tokens.iter().enumerate() {
    if let Some(postings) = self.get_term_postings(token) {
        let nq = self.vocabulary.get_term_frequency(token).unwrap() as f64;
        let idf = ((n - nq + 0.5) / (nq + 0.5) + 1.0).ln();

        for doc_posting in &postings {
            let fq = doc_posting.document_frequency as f64;
            let dl = self.documents.get_doc_len(doc_posting.document_id) as f64;

            let bm_score = idf * (fq * (BM25_KL + 1.0))
                / (fq + BM25_KL * (1.0 - BM25_B + BM25_B * (dl / avgdl)));

            let doc_score = scores.entry(doc_posting.document_id).or_default();
            doc_score.tf_idf += bm_score;
        }
    }
}
```

**Window score**

After obtaining the BM25 score, we also compute the minimum window in which the query terms appear in a document, setting it at infinite if not all terms appear in the same corpus. For instance, given a query *gun control*, finding *gun and control* in a document would result in a size 3 window.

$$\text{window}(D, Q) = \frac{|Q|}{\text{min. window}(Q, D)}$$

**Final rank function**

The final rank function is then: 

$$\text{score}(D, Q) = \alpha \cdot \text{window}(D, Q) + \beta \cdot \text{BM25}(D, Q)$$

The window and BM25 scores are **relevance signals**, a production search engine would many more, such as document quality, [PageRank](https://en.wikipedia.org/wiki/PageRank) scoring, etc. The weights to combine them could be learned with a machine learning model, trained on a doc-query pair dataset.

### Spelling correction

The final aspect we are going to see about queries is spelling correction.
The idea is to edit user input and replace unknown words with plausible ones. 
To measure words similarity we can use [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance), also knows as edit distance, 
counting the minimum needed operations to transform a string into another one, 
performing insertion, deletion, and substitutions.

We can compute it efficiently with dynamic programming, using the following 
definition.

$$
\text{lev}(a, b) = \begin{cases}
    |a| & \text{if}\\\;|b| = 0, \\\\
    |b| & \text{if}\\\;|a| = 0, \\\\
    1 + \text{min} \begin{cases}
        \text{lev}(\text{tail}(a), b) \\\\
        \text{lev}(a, \text{tail}(b)) \\\\
        \text{lev}(\text{tail}(a), \text{tail}(b)) \\\\
    \end{cases} & \text{otherwise} \\\\
\end{cases}
$$

To avoid running the function on every term in the vocabulary to 
find the one with minimum distance, we can restrict heavily the candidates 
using a [trigram index](https://en.wikipedia.org/wiki/Trigram_search) on the terms.

For instance, given the word *hello* we search for words containing the 
trigrams *hel*, *ell*, and *llo*. We therefore keep an index 
in-memory where for every trigram occurring in the vocabulary, we 
have references to terms.

When we find a tie in edit distance, we prefer higher overall frequency.

```rust
fn get_closest_index(&self, term: &str) -> Option<usize> {
    // "hello" -> "hel", "ell", "llo"
    let candidates = (0..term.len() - 2)
        .map(|i| term[i..i + 3].to_string())
        .filter_map(|t| self.trigram_index.get(&t))
        .flat_map(|v| v.iter());

    // min edit distance, max frequency
    candidates
        .min_by_key(|i| {
            (
                Self::levenshtein_distance(term, &self.index_to_term[**i]),
                -(self.frequencies[**i] as i32),
            )
        })
        .copied()
}
```

## Writing data on disk

To avoid having to recompute the index every time, we store it on disk. 
We need four files: 
1. *Postings*: it contains all the term-document pairs, with frequency information and term positions; 
2. *Offsets*: offset for term *i* in bits in the postings;
3. *Alphas*: complete vocabulary with document frequency information;
4. *Docs*: info about documents, such as the disk path and length.

**Postings format**

For each term we save a postings list as follows: 
$$\text{n}\\;|\\;(\text{id}_i, f_i, [p_0, \dots, p_m]), \dots$$

Where $n$ is the number of documents the term appears in, id is the 
doc id, $f$ is the frequency, and $p_j$ are the positions where 
the term appears in the document $i$.

**Offsets format**

The offset file is a sorted sequence of bits offsets.
$$\text{n}\\;|\\;o_0, \dots, o_n$$

**Alphas**

A list of sorted words, with their collection frequency.
$$\text{n}\\;|\\;w_0, f_0, \dots, w_n, f_n$$

**Docs**

A list of documents paths, with their length.
$$\text{n}\\;|\\;p_0, l_0, \dots, p_n, l_n$$

**Space occupation**

Here is how much memory an index for ~180k documents with 32-bit integers representation takes on disk: 

| Type       | Size MB |
| ---------- | ------: |
| Vocabulary |     5.4 |
| Documents  |     7.2 |
| Offsets    |     1.1 |
| Postings   |  2200.0 |

### Exploiting small integers

We can do way better, exploiting the distribution of the integers we write. 
The postings list document ids are strictly increasing, 
as documents are sorted, hence if we use delta encoding we obtain smaller integers.
The same goes for term positions and the entire offset file.

We could therefore store gaps instead of the entire numbers, this creates many small integers, 
hence we can use something 
like [Gamma encoding](https://en.wikipedia.org/wiki/Elias_gamma_coding), which 
uses few bits for small numbers. 

The idea is to take the binary 
representation of an integer and write its length in unary before it.
For instance, $5$, and its binary representation $101$, will then 
be written as $001$, concatenated with $101$, we can merge the two center ones
to avoid wasting one bit $00101$. Here are the first integers:

$$
\begin{align*}
1 &\rightarrow  1 \\\\
2 &\rightarrow  010 \\\\
3 &\rightarrow  011 \\\\
4 &\rightarrow  00100 \\\\
5 &\rightarrow  00101 \\\\
6 &\rightarrow  00110 \\\\
7 &\rightarrow  00111 \\\\
&\dots
\end{align*}
$$

For generic integers, such as list lengths, we can use [VByte encoding](https://nlp.stanford.edu/IR-book/html/htmledition/variable-byte-codes-1.html). I already mentioned it in my other blog post about [LSM-trees](/posts/lsm/#data-layout), go have a look if you like.
The idea is similar, we split an integer into 7-bit payloads and use the remaining bit to indicate whether the payload continues or not. Integers would then use one to four bytes, instead of 32 bits every time.

### Using words prefixes

Vocabulary is also sorted, hence we can use prefix compression to store the terms. Take for instance *watermelon*, *waterfall*, and *waterfront*. Instead of writing them as they are, we store the length of the matching prefix with the previous word, followed by the remaining suffix. 

The naïve representation: 
$$\text{watermelon}\\;\text{waterfall}\\;\text{waterfront}$$

Then becomes: 
$$0\\;\text{watermelon}\\;5\\;\text{fall}\\;6\\;\text{ront}$$

We can apply the same principles with document paths, as they likely share directories.

### Final representation

After employing prefix compression and delta encoding with VByte and Gamma codes, we save over **~68%** of disk space compared to the naïve representation.

| Type       | Uncompressed (MB) | Compressed (MB) | Saving Ratio (%) |
| ---------- | ----------------: | --------------: | ---------------: |
| Vocabulary |               5.4 |             1.3 |             75.9 |
| Documents  |               7.2 |             2.3 |             68.1 |
| Offsets    |               1.1 |             0.6 |             46.5 |
| Postings   |            2200.0 |           724.0 |             67.1 |

### Implementation details

The project defines a writer and reader to store those codes on disk. Although not trivial to read, the idea is to use a buffered writer and reader to interact with the disk, and a 128-bit long as a temporary buffer on top of it. 

When we want to write a given integer, we first build a binary payload containing its Gamma representation and then append it to the bit buffer via bit manipulation.
Once the buffer reaches 128 bits, it is flushed to the underlying buffered writer.

```rust
pub fn write_gamma(&mut self, n: u32) -> u64 {
    let (gamma, len) = BitsWriter::int_to_gamma(n + 1);
    self.write_internal(gamma, len)
}

fn int_to_gamma(n: u32) -> (u128, u32) {
    let msb = 31 - n.leading_zeros();
    let unary: u32 = 1 << msb;
    let gamma: u128 = (((n ^ unary) as u128) << (msb + 1)) | unary as u128;
    (gamma, 2 * msb + 1)
}

fn write_internal(&mut self, payload: u128, len: u32) -> u64 {
    let free = 128 - self.written;
    self.buffer |= payload << self.written;

    if free > len {
        self.written += len;
    } else {
        self.update_buffer();
        if len > free {
            self.buffer |= payload >> free;
            self.written += len - free;
        }
    }

    len as u64
}
```

## Web client

To have a nicer interaction with the engine I made a simple web interface using 
[Actix](https://actix.rs/), [HTMX](https://htmx.org/) and [Askama](https://github.com/djc/askama) templates.

You can load an existing index and query it with free or boolean queries.

![client](web-l.webp#light "Light mode Web client showing a boolean query")
![client](web-d.webp#dark "Dark mode Web client showing a boolean query")

## Conclusions

Overall, this was a fun project, I saw many new concepts and more importantly, it got me started with Rust. I would appreciate any comment about the source code, as this was my first time with the language.

Thank you for reading this far, feel free to get in touch for for suggestions or clarifications, if you found this interesting, here is my [previous article](/posts/lsm/) about LSM Trees, have a look!

Have a nice day 😃

### References
- [Introduction to Information Retrieval](https://nlp.stanford.edu/IR-book/information-retrieval-book.html)