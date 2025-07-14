---
title: "1111 my tests"
# author: "Rostyslav Ivanitsa"
# authorAvatarPath: "/avatar.jpeg"
date: "2025-07-05"
summary: "1111 A "
description: "What are the communication bottlenecks between us and it?"
toc: true
readTime: true
autonumber: true
math: true
tags: ["rust"]
showTags: true
---

I've been in the process of coding a web crawler in Rust and faced a problem with keeping an in-memory to-visit queue.
The idea is to have a collection of nodes to visit, which discards duplicates, with minimal memory overhead.

## The Problem

A problem like this could be solved by using a simple set. Before inserting a new element, we check for membership into the visited set and discard duplicates.

In the following examples, rust is used, and elements are Strings, coming from crawled web pages.

```rust
pub struct Sieve {
    filter: HashSet<String>,
    urls: VecDeque<String>,
}

impl Sieve {
    pub fn new() -> Sieve {
        Sieve {
            filter: HashSet::new(),
            urls: VecDeque::new(),
        }
    }

    pub fn push(&mut self, url: String) {
        if self.filter.contains(&url) {
            return;
        }

        self.filter.insert(url.clone());
        self.urls.push_back(url);
    }

    pub fn pop(&mut self) -> Option<String> {
        self.urls.pop_front()
    }
}
```
This should be fine right? The answer is yes if the elements you add to it are limited, otherwise, the memory consumption of the set grows indefinitely.

We can do better by relying on the same ideas but with a different Set implementation.

## Bloom Filters

A Bloom Filters is a space-efficient probabilistic data structure. It can be used to answer membership queries with one caveat, it makes errors with a certain probability.

### The Algorithm

An Bloom Filter is composed by two elements: 
- A bit array of $m$ bits, initially set to zero;
- $d$ hash functions, where $d$ is a small constant.

**Adding an element**

To add a new element, we feed it to the $d$ hash functions, obtaining $d$ indexes in the range $[0, m-1]$, used to set these positions to one in the bit array.

**Checking for membership**

As for insertion, we compute $d$ hash values, and then check if all those bits are set to one in the bit array. 

### Dealing with False Positives

A Filter is constructed with two parameters in mind: the number of expected insertions, and the wanted false-positive rate. Those two quantities determine the required number of bits and hash functions in the structure.

Given $n$ the number of wanted elements, and $\epsilon$ the wanted false positive rate, the following holds: 
$$ m = -\frac{n * \ln(p)}{\ln(2)^2} $$

The number of hash functions can be computed as:
$$ d = -\frac{\ln(p)}{\ln(2)} $$

As those relations hold, we can balance the tradeoffs between speed, as more hash functions mean more bits to set and false-positive rate.

### Practical Hash Computation

We do not want to compute an excessive number of hash functions, as they can be quite costly. An easy way to reduce the required number of hash computations is to compute two hash values and combine them like so: 

$$ h_i = h_1 + h_2 * i$$

To compute two different hash values we can hash an element once and split the hash bits in two.

### Implementation

Here is a practical implementation of the Bloom Filter. To represent a bit array we use a vector of 128 bit numbers where each bit is considered as a different position.

```rust
pub struct Filter {
    size: usize,
    d: u32,
    bits: Vec<u128>,
    set_bits: u32,
}
```

**Construction**

Given the expected number of insertions and false-positive rate, we compute the optimal number of bits to use.

```rust
pub fn new(n: usize, p: f64) -> Filter {
    let log_2 = 2_f64.ln();
    let log_p = p.ln();

    let size = ((-(n as f64) * log_p) / (log_2 * log_2)) as usize;
    let d = (-log_p / log_2).ceil() as u32;
    let bits = vec![0; (size as f64 / 128.0).ceil() as usize];
    let set_bits = 0;

    Filter {
        size,
        d,
        bits,
        set_bits,
    }
}
```

**Insertion**

We compute $d$ hash values and use them to set the corresponding bits in the bit array.

```rust
pub fn add(&mut self, data: &[u8]) {
    let (h1, h2) = Self::hash(data);

    for i in 0..self.d {
        let bit = (h1 as u128 + h2 as u128 * i as u128) as usize % self.size;
        self.bits[bit / 128] |= 1 << (bit % 128);
    }
}
```

**Membership Queries**

We compute $d$ hash values and use them to check that the bits are all set.

```rust
pub fn contains(&mut self, data: &[u8]) -> bool {
    let (h1, h2) = Self::hash(data);

    for i in 0..self.d {
        let bit = (h1 as u128 + h2 as u128 * i as u128) as usize % self.size;
        if self.bits[bit / 128] & (1 << (bit % 128)) == 0 {
            return false;
        }
    }
    true
}
```

**Estimating size**

Size can be estimated with a simple formula: 

$$s = - \frac{m}{d} \ln \Bigg [ 1 - \frac{X}{m} \Bigg ] $$

Where $X$ is the number of bits set to one.

```rust
pub fn estimated_size(&self) -> usize {
    (-(self.size as f64 / self.d as f64)
        * (1f64 - self.set_bits as f64 / self.size as f64).ln()) as usize
}
```

We hence need a new value, `set_bits`, which can be updated on each insertion. Here is the tweaked `add` function.

```rust
pub fn add(&mut self, data: &[u8]) {
    ...
    // subtract old number of set bits in this block
    self.set_bits -= self.bits[bit / 128].count_ones();
    // set bit to one
    self.bits[bit / 128] |= 1 << (bit % 128);
    // add new number of set bits in this block
    self.set_bits += self.bits[bit / 128].count_ones();    
    ...
}
```

**Full Implementation**

Wrapping everything up, here is the complete Bloom Filter implementation.

```rust
pub struct Filter {
    size: usize,
    d: u32,
    bits: Vec<u128>,
    set_bits: u32,
}

impl Default for Filter {
    fn default() -> Self {
        Self::new(1_000_000, 0.01)
    }
}

impl Filter {
    pub fn new(n: usize, p: f64) -> Filter {
        let log_2 = 2_f64.ln();
        let log_p = p.ln();

        let size = ((-(n as f64) * log_p) / (log_2 * log_2)) as usize;
        let d = (-log_p / log_2).ceil() as u32;
        let bits = vec![0; (size as f64 / 128.0).ceil() as usize];
        let set_bits = 0;

        Filter {
            size,
            d,
            bits,
            set_bits,
        }
    }

    pub fn add(&mut self, data: &[u8]) {
        let (h1, h2) = Self::hash(data);

        for i in 0..self.d {
            let bit = (h1 as u128 + h2 as u128 * i as u128) as usize % self.size;
            self.set_bits -= self.bits[bit / 128].count_ones();
            self.bits[bit / 128] |= 1 << (bit % 128);
            self.set_bits += self.bits[bit / 128].count_ones();
        }
    }

    pub fn contains(&mut self, data: &[u8]) -> bool {
        let (h1, h2) = Self::hash(data);

        for i in 0..self.d {
            let bit = (h1 as u128 + h2 as u128 * i as u128) as usize % self.size;
            if self.bits[bit / 128] & (1 << (bit % 128)) == 0 {
                return false;
            }
        }
        true
    }

    pub fn estimated_size(&self) -> usize {
        (-(self.size as f64 / self.d as f64)
            * (1f64 - self.set_bits as f64 / self.size as f64).ln()) as usize
    }

    fn hash(data: &[u8]) -> (u64, u64) {
        let h = fastmurmur3::hash(data);
        let mask: u128 = (1 << 64) - 1;
        ((h & mask) as u64, (h >> 64) as u64)
    }
}
```

### Memory Footprint

A great property of those Filters is that their memory footprint is fixed, independent of how many elements we store. What degrades is the false positive rate, until it reaches one. What degrades is the false positive rate, until it reaches one. 

The space efficiency is amazing, for instance, **1 Billion elements**, with a false positive rate of **0.01%** can be stored in just over **18mb**.

## The Solution

Here is the final solution to this problem. The Sieve struct now accepts a parameter indicating the expected number of elements that are enqueued. Once we surpass this threshold, the filter performance starts to deteriorate.

```rust
use log::warn;

pub struct Sieve {
    filter: Filter,
    urls: VecDeque<String>,
    expected_size: usize,
}

impl Sieve {
    pub fn new(expected_urls_num: usize) -> Sieve {
        Sieve {
            filter: Filter::new(expected_urls_num, 0.01),
            urls: VecDeque::new(),
            expected_size: expected_urls_num,
        }
    }

    pub fn push(&mut self, url: String) {
        let url_bytes = url.as_bytes();

        if self.filter.contains(url_bytes) {
            return;
        }

        self.filter.add(url_bytes);
        self.urls.push_back(url);

        let filter_size = self.filter.estimated_size();
        if filter_size >= self.expected_size {
            warn!(
                "Filter size ({}) exceeds Sieve expected size ({})",
                filter_size, self.expected_size
            );
        }
    }

    pub fn pop(&mut self) -> Option<String> {
        self.urls.pop_front()
    }
}
```

Thank you for reading this far, feel free to get in touch for suggestions or clarifications!
Have a nice day 😃

