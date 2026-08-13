---
layout: post
title: "30× faster binary multivector ColBERT late interaction in Vespa"
categories: engineering
math: true
---

A key advantage of Vespa against other search databases is the ease with which you can index multiple chunks of a long document in a multi-vector fashion. This is convenient in RAG applications as it allows your overall document score to be an arbitrary score of the individual chunk scores.

This is true, even if the individual chunk tensors themselves are two dimensional token vector embeddings. In such a setting, a document is a 3-dimensional tensor queried by a 2-dimensional query tensor of token vectors. While similar functionality is offered e.g. by Elasticsearch's [`nested`](https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/nested) field type, only Vespa offers fully general tensor machinery.


## Maximum similarity

The similarity between ColBERT late interaction embeddings of a query and a chunk is the sum of the maximum similarities of each query token with respect to the most similar document token within the chunk. This is commonly called *MaxSim*.

Let $$q$$ identify a query token, $$t$$ a document token, and $$c$$ a document
chunk, and let $$d_{c,t}$$ be the embedding of token $$t$$ in chunk $$c$$. If
$$s(q,d_{c,t})$$ is the (e.g. cosine) similarity between the two token vectors, then the score of
one chunk is

$$
S_c = \sum_q \max_t s(q,d_{c,t})
$$

Our similarity of the whole document with respect to the query is the maximum of its chunk similarities:

$$
S = \max_c S_c
  = \max_c \sum_q \max_t s(q,d_{c,t})
$$

In other words, **one chunk** within the document must answer the whole query. Query tokens may choose different document tokens, but all of
those tokens must come from the same chunk. 

## Hamming maximum similarity

At least since [ColBERTv2](https://arxiv.org/pdf/2112.01488), almost everyone has been compressing their embeddings to deal with the storage and compute demands of storing an embedding vector for each document token at scale. We go all the way to binary: in our schema each 128-dimensional ColBERT document vector is packed into 16 bytes. Inverse Hamming distance is a similarity metric between binary vectors that is fast to compute:

$$
s_H(q,d)
= \frac{1}{1+H(q,d)}
$$

This reciprocal is strictly decreasing in $$H$$, so within a chunk each query token still selects the document token with the smallest Hamming distance. It also heavily rewards almost identical bit patterns and compresses the rest of the curve:

![Normalized binary dot product compared with inverse Hamming similarity for
32-, 128-, and 512-bit vectors. The inverse-Hamming curves fall increasingly
sharply as vector length grows.](/assets/diagrams/hamming-similarity-curves.svg)

At one differing bit, the inverse Hamming distance has already fallen by half. Longer vectors make the reciprocal sharper because the function literally just counts bits. With inverse Hamming distance, the document MaxSim equation is

$$
S
= \max_c \sum_q \max_t \frac{1}{1 + H(q,d_{c,t})}.
$$

## Optimization

A useful transformation comes from the fact that inverse distance is
monotonically decreasing. Within a chunk, maximizing inverse distance is the
same as minimizing distance first:

$$
S = \max_c \sum_q \frac{1}{1 + \min_{t} H(q,d_{c,t})}.
$$

This expression gives us the blueprint for a fast algorithm:

1. Allocate one minimum distance for every `(chunk, query token)` pair.
2. Walk the document vectors once.
3. Compare each document vector with every query vector and update the
   corresponding minimum.
4. Convert the minima to inverse-distance scores and sum them per chunk.
5. Return the largest chunk score.

In [call stack diff](https://oskrim.github.io/engineering/2026/08/02/call-stack-diffs.html) notation, the new algorithm is roughly:

```diff
 Ranking expression
   -> optimize_tensor_function(...)
-      -> generic implementation
-          -> materialize query × chunk × document-token intermediates
+      -> MaxSumMaxInvHammingFunction::optimize(...)
+          -> my_max_sum_max_inv_hamming_op(...)
+              -> one pass over the document token vectors
+              -> minimum distance per chunk × query token
+                  -> exact match: skip the remaining comparisons
+              -> sum per chunk
+              -> maximum chunk score 
```

The implementation is available in [a pull request on
Vespa](https://github.com/vespa-engine/vespa/pull/37563). The fast path deliberately
matches this exact shape:

```text
query:    tensor<int8>(qt{},x[N])
document: tensor<int8>(chunk{},t{},x[N])
max(chunk, sum(qt, max(t, 1 / (1 + sum(x, hamming(query, document))))))
```

Both tensors must use `int8` cells, the document dimensions must be mapped,
and the reductions must have this order. Without these tensor and expression shapes, the optimization will not trigger.

## Result

I measured a small dataset of 16-byte binary vectors, 32 query vectors, and 301 chunks of 138 document vectors:

| Measurement | Result |
|:--|--:|
| Before | 91.8 ms |
| After | 2.8 ms |
| Speedup | **32.8×** |

The 32.8x speedup is roughly in line with that [observed for the two-dimensional optimization](https://github.com/vespa-engine/vespa/issues/32232#issuecomment-2367477852).

At [Realm](https://www.withrealm.com/blog/seed-round), we have been running the new patch live in production for thousands of users for a few days already, observing 10x speedups for queries that used to be unbearably slow. If you are interested in working with information retrieval techniques and context engineering, [we are hiring!](https://realmtechnologies.notion.site/AI-Engineer-1c0bddf3cd2e80c68a83d30e166df39d?pvs=4)
