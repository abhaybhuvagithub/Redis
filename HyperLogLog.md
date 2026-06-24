# Redis HyperLogLog - Complete Guide

## Overview
Independent Analysis By Abhaykumar Bhuva

**HyperLogLog (HLL)** is a probabilistic data structure used to estimate the number of unique elements (cardinality) in a dataset while consuming a very small and fixed amount of memory.

Unlike a Redis Set, HyperLogLog does not store every element. Instead, it uses advanced mathematical algorithms to estimate the count of distinct values.

---

# Why HyperLogLog?

Imagine tracking unique website visitors.

## Using Redis Sets

```text
1,000,000,000 visitors × 16 bytes
≈ 16 GB+
```

The actual memory consumption is even higher because Redis Sets have internal overhead.

## Using HyperLogLog

```text
≈ 12 KB
```

Whether you process:

* 1 Million users
* 10 Million users
* 1 Billion users
* 100 Billion users

Memory remains approximately:

```text
12 KB
```

---

# What is Cardinality?

Cardinality means:

> The number of distinct elements in a collection.

Example:

```text
A
B
C
A
B
D
```

Unique values:

```text
A
B
C
D
```

Cardinality:

```text
4
```

HyperLogLog estimates this value efficiently.

---

# Redis HyperLogLog Commands

Redis provides three commands:

```redis
PFADD
PFCOUNT
PFMERGE
```

PF stands for Probabilistic Function.

---

# PFADD

Adds elements to a HyperLogLog.

```redis
PFADD visitors user1 user2 user3
```

Output:

```text
(integer) 1
```

Meaning the estimated cardinality changed.

Adding duplicates:

```redis
PFADD visitors user1
```

Output:

```text
(integer) 0
```

Meaning the estimate remained unchanged.

---

# PFCOUNT

Returns the estimated cardinality.

```redis
PFCOUNT visitors
```

Output:

```text
3
```

Example:

```redis
PFADD visitors user1 user2 user3 user4
PFCOUNT visitors
```

Output:

```text
4
```

---

# PFMERGE

Merges multiple HyperLogLogs.

```redis
PFMERGE all_users us_users eu_users asia_users
```

Get total unique users:

```redis
PFCOUNT all_users
```

---

# Example

Website A:

```redis
PFADD websiteA u1 u2 u3
```

Website B:

```redis
PFADD websiteB u2 u3 u4
```

Merge:

```redis
PFMERGE total websiteA websiteB
```

Count:

```redis
PFCOUNT total
```

Result:

```text
4
```

Not:

```text
6
```

because duplicates are automatically handled by the algorithm.

---

# How HyperLogLog Works Internally

## Step 1: Hash Every Input

Input:

```text
user123
```

Redis converts it into a 64-bit hash.

Example:

```text
001101010101011001...
```

---

## Step 2: Split Hash Into Registers

Redis HyperLogLog contains:

```text
16384 Registers
```

Because:

```text
2^14 = 16384
```

The first 14 bits determine which register will be used.

Example:

```text
00110101010101
```

May correspond to:

```text
Register #3413
```

---

## Step 3: Count Leading Zeros

Remaining bits are analyzed.

Example:

```text
00000000000110101
```

Leading zeros:

```text
11
```

This is important because long runs of leading zeros are statistically rare.

The rarer the pattern observed, the larger the estimated population.

---

# Register Updates

Each register stores:

```text
Maximum leading-zero count observed
```

Example:

```text
Current Value = 5
New Value = 8
```

Store:

```text
8
```

Because:

```text
max(5,8) = 8
```

---

# Cardinality Estimation

After processing all elements:

```text
16384 registers
```

contain values such as:

```text
2
4
5
3
6
...
```

HyperLogLog then applies:

* Harmonic Mean
* Statistical Corrections
* Bias Corrections

to generate the final estimate.

Example:

```text
987,654,321 unique users
```

---

# Why Only 12 KB?

Each register stores:

```text
6 bits
```

Number of registers:

```text
16384
```

Calculation:

```text
16384 × 6
= 98,304 bits
≈ 12 KB
```

Therefore memory remains nearly constant regardless of dataset size.

---

# Accuracy

Redis HyperLogLog has a standard error of approximately:

```text
0.81%
```

Example:

Actual:

```text
10,000,000
```

Estimated:

```text
9,919,000
```

or

```text
10,081,000
```

For analytics systems, this level of error is usually acceptable.

---

# HyperLogLog vs Redis Set

| Feature            | Redis Set | HyperLogLog |
| ------------------ | --------- | ----------- |
| Exact Count        | ✅ Yes     | ❌ No        |
| Unique Count       | ✅ Yes     | ✅ Estimated |
| Membership Check   | ✅ Yes     | ❌ No        |
| Memory Usage       | High      | ~12 KB      |
| Billion Elements   | Difficult | Easy        |
| Supports Removal   | ✅ Yes     | ❌ No        |
| Analytics Friendly | Limited   | Excellent   |

---

# When to Use HyperLogLog

Use HyperLogLog for:

* Website visitor analytics
* Daily Active Users (DAU)
* Monthly Active Users (MAU)
* Unique API consumers
* Unique IP tracking
* IoT device counting
* Advertisement analytics
* Fraud detection metrics
* Distributed analytics systems

---

# When NOT to Use HyperLogLog

Avoid HyperLogLog when:

* Exact counts are required
* Membership checks are required
* Individual elements must be retrieved
* Elements need to be removed
* Set operations are needed

In these cases, use Redis Sets instead.

---

# Redis Internal Representations

Redis uses two storage modes.

## Sparse Representation

Used when few elements exist.

Advantages:

* Lower memory usage
* Compact storage

---

## Dense Representation

Automatically activated as cardinality grows.

Memory:

```text
~12 KB
```

Fixed-size structure optimized for large datasets.

---

# Production Use Cases

Large-scale systems commonly use HyperLogLog for:

* User analytics
* Network telemetry
* Advertising systems
* Big Data pipelines
* Event processing systems
* Observability platforms

---

# Interview Questions

## Why use HyperLogLog instead of a Set?

HyperLogLog provides cardinality estimation using approximately 12 KB of memory, while Sets require memory proportional to the number of elements.

---

## Can HyperLogLog determine if `user123` exists?

No.

HyperLogLog only estimates the number of unique elements and does not store actual values.

---

## Can elements be removed?

No.

Redis HyperLogLog only supports adding and counting.

---

## What is the typical error rate?

Approximately:

```text
0.81%
```

---

## How much memory does Redis HyperLogLog consume?

Approximately:

```text
12 KB
```

per HyperLogLog key in dense mode.

---

# Key Takeaways

* HyperLogLog is a probabilistic data structure.
* Designed for cardinality estimation.
* Uses approximately 12 KB of memory.
* Error rate is around 0.81%.
* Ideal for large-scale analytics workloads.
* Supports `PFADD`, `PFCOUNT`, and `PFMERGE`.
* Trades perfect accuracy for massive memory savings.

---

## References

https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/
* Redis HyperLogLog Documentation
* Flajolet HyperLogLog Research Paper
* Redis Source Code
* Distributed Systems Cardinality Estimation Techniques
