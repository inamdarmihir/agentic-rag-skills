---
name: qdrant-payload-indexing
description: |
  Use when filtered search is slow in Qdrant, deciding which payload fields to index,
  payload filter performance is degrading at scale, choosing between keyword, integer,
  float, and geo index types, a filter is scanning all points instead of using an index,
  debugging slow filtered queries, cardinality of a payload field is affecting index choice,
  or combining payload filters with vector search efficiently.
---

# Payload Indexing in Qdrant

Qdrant evaluates payload filters during HNSW graph traversal, not after. An indexed field prunes the traversal early; an unindexed field scans every candidate point. At 1M+ points, unindexed filters are the most common cause of slow filtered search.

## Which fields to index

Use when: deciding what to index before ingesting data.

- Index every field you use in `must`, `should`, or `must_not` filter conditions
- Prioritize fields that appear in filters that discard the most results; a `tenant_id` filter that keeps 1% of points benefits from an index far more than a `rating` filter that keeps 80%
- Do not index fields you only store for display purposes (e.g., `display_name`, `raw_text`); indexes consume memory proportional to cardinality — [qdrant.tech/documentation/concepts/indexing](https://qdrant.tech/documentation/concepts/indexing/)
- Index fields you filter on most frequently, even at low cardinality; keyword indexes on boolean-like fields (3–5 distinct values) are small and fast

## Index types

Use when: choosing the correct index type for a payload field.

- **Keyword index**: for string fields with categorical values (status, type, tenant_id, language); efficient for equality and membership filters
- **Integer index**: for integer fields used in range filters (year, version_number, count); supports `range`, `gt`, `gte`, `lt`, `lte`
- **Float index**: for float fields in range filters (score, price, confidence); same operations as integer index
- **Datetime index**: for ISO 8601 date strings or Unix timestamps stored as integers; use integer storage + integer index for simplicity unless you need human-readable payload values
- **Geo index**: for latitude/longitude coordinates; enables `geo_radius` and `geo_bounding_box` filters — [qdrant.tech/documentation/concepts/payload](https://qdrant.tech/documentation/concepts/payload/)
- **Text index**: for full-text search within payload fields (not the same as sparse vectors); use for substring or phrase matching within long text fields

## Compound filters and index interaction

Use when: combining multiple filter conditions.

- Qdrant evaluates `must` conditions with AND logic; it uses the most selective indexed condition to prune candidates first
- Put the most selective condition first in your `must` array; Qdrant processes conditions in order and short-circuits early
- Avoid combining indexed and unindexed fields in the same filter; the unindexed field nullifies the performance benefit of the indexed one for that filter path
- Use `should` (OR) carefully at scale; OR conditions expand the candidate set rather than pruning it

## Index cardinality and memory

Use when: payload indexes are consuming too much memory.

- High-cardinality keyword fields (UUIDs, email addresses, free-form strings) produce large keyword indexes with minimal filter selectivity; store these as payload but do not index unless you have a specific filter need
- Integer and float indexes have fixed memory overhead per unique value; they scale better than keyword indexes on high-cardinality numeric fields
- Check index memory usage per field by inspecting collection info — [qdrant.tech/documentation/concepts/collections](https://qdrant.tech/documentation/concepts/collections/)
- Drop indexes on fields that are no longer used in filters; dangling indexes consume memory and slow upserts

## Adding indexes to existing collections

Use when: adding an index after initial data ingestion.

- Qdrant builds payload indexes on existing data in the background; queries during index build may be slower for the indexed field while it completes
- Index creation on large collections can take minutes; trigger it during low-traffic periods
- Verify index creation completed before running production filtered queries; an in-progress index does not yet provide the full performance benefit

## Debugging slow filtered queries

Use when: a filtered search is taking significantly longer than an unfiltered search.

- Run `GET /collections/{collection_name}` and check `payload_schema` to verify indexes exist for the fields in your filter
- Compare filtered vs. unfiltered search latency on the same collection; if filtered is 10x slower, the filter field is likely unindexed
- For very high-selectivity filters (>90% of points excluded), add the filter field as a custom shard key to co-locate filtered data — [qdrant.tech/documentation/guides/distributed-deployment](https://qdrant.tech/documentation/guides/distributed-deployment/)

## What NOT to Do

- Do not filter on unindexed fields in collections larger than 100k points; it will full-scan on every query
- Do not create keyword indexes on UUID fields or other fields with one unique value per point; the index will be as large as the collection with no filter benefit
- Do not use text indexes as a replacement for sparse vectors; text indexes are for payload substring matching, not semantic or BM25 retrieval
- Do not add indexes speculatively "just in case"; indexes consume memory on every upsert and slow write throughput
