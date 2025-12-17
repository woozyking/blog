---
title: 'Refactor for Sustainability: Reducing Our Data Pipeline Footprint (Part 1)'
date: 2025-12-16 16:34:34
tags:
  - Data Engineering
  - Sustainability
  - Polars
---

## **TL;DR**

We rewrote our asset preprocessing pipeline, replacing a "Frankenstein" mix of Pandas and manual multiprocessing with an end-to-end **Polars LazyFrame** workflow. The result? A **6x speedup** and a massive reduction in memory usage (from **110GB** to **2GB**). By optimizing for efficiency, we didn't just save money; we aligned our engineering capability with our mission: doing more with less to reduce our impact.

![simplify-data-pipeline](/images/simplify-data-pipeline.webp)

## **The Cost of Digital Waste**

Optimization is often framed as a cost-saving exercise or a speed run. But at [**RiskThinking.AI**](http://riskthinking.ai/), we view it through a slightly different lens. We analyze climate risk, helping the world understand the impact of a changing environment. It felt philosophically inconsistent for us to be generating unnecessary "digital exhaust" in our own backyard.

Our asset ingestion pipeline was reliable, but it was heavy. As our datasets grew to millions of assets, our legacy code required massive cloud instances to keep running. We were burning compute cycles, and by extension, **energy**, to solve a problem that shouldn't have been that hard.

We decided to rebuild it, not just to go faster, but to stop the waste.

<!-- more -->

## The "Frankenstein" Bottleneck

Our original implementation suffered from a common pattern: trying to micromanage performance.

To process millions of rows, we were:

1. Manually chunking data.
2. Spinning up a `multiprocessing.Pool`.
3. Materializing intermediate results in memory before concatenating them.

It looked something like this:

```python
# The Old Way: Manual chunking and multiprocessing
with mp.get_context("spawn").Pool(processes=n_cpu) as pool:
    args = [(chunk, assets, top_naics) for chunk in chunks]
    results = pool.starmap(process_chunk_for_materiality, args)

df = pd.concat(results)
df.write_parquet(output_path)
```

This approach is fine for smaller tasks, but at scale, the overhead of context switching and memory duplication eats up all the gains. We needed something that could handle the volume without the manual labour and bloated computing cost.

## Be Lazy (The Good Kind)

We replaced the entire flow with **Polars**. The key wasn't just swapping libraries (i.e., `pd.read_parquet` for `pl.read_parquet`), but embracing the **Lazy API**.

Pandas executes eagerly. Every step materializes intermediate results in RAM, forcing you to rent larger servers just to hold temporary data. Polars operates lazily. Operations like `scan_parquet` build a query plan. Polars then runs a query optimizer across your ETL operations to apply **predicate pushdown** (filtering data at scan time), **projection pushdown** (loading only necessary columns), and [**more**](https://docs.pola.rs/user-guide/lazy/optimizations/).

The magic happens when you call `.sink_parquet()`. This triggers Polars' **streaming engine**, which processes data in batches that fit in RAM and parallelizes across available CPU cores. By decoupling memory usage from dataset size, you can run "big data" jobs on "small hardware."

```python
# The New Way: Declarative, lazy, and streaming
(
    pl.scan_parquet(input_path)
    .filter(pl.col("materiality_score") >= THRESHOLD)
    .pipe(utils.apply_materiality_scores, mat_score_dict)
    .sink_parquet(output_path) # The trigger: streams data from input to output
)
```

## Surgical Materialization

The only snag was our dependency on GeoPandas for spatial joins (mapping lat/lon to cities/states.)

Instead of reverting to eager execution for everything, we adopted a "surgical" approach:

1. Keep the main pipeline lazy.
2. Filter specifically for the subset of rows missing geo data.
3. `.collect()` (materialize) *only* that subset into RAM.
4. Run the heavy GeoPandas operations.
5. Convert back to Polars and merge.

**This kept our memory profile reasonable**, even when processing millions of assets. *In a future follow-up, we'll detail how we engineered a solution to eliminate this materialization step entirely, making the pipeline 100% lazy.*

## Results: Efficiency is Green

We compared our legacy pipeline running on the cloud against the new pipeline running on a local machine.

- **The Old Way:** Required **15 vCPUs and 110GB of RAM** (Cloud). Time: ~a hour.
- **The New Way:** Ran on a **MacBook Pro (M1, 16GB RAM)**. Time: ~10 mins. Peak RAM: **2GB**.

We eliminated the need for a massive, expensive node. A job that used to require a semi-truck now runs on a bicycle.

## Final Remark

Every real-world technique comes with tradeoffs. But for us, the shift to Polars wasn't just about the 6x speedup. It was about right-sizing our infrastructure.

When you write code that is 98% more memory efficient, you aren't just making your life easier as a developer. You are reducing the demand on data centers and lowering the energy cost of your insights. That’s a win for the product, and a small but meaningful win for the planet.