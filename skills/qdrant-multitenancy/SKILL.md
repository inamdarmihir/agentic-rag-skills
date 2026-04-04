---
name: qdrant-multitenancy
description: |
  Use when building a multi-tenant application on Qdrant, isolating data between customers,
  one Qdrant collection needs to serve multiple users or organizations, deciding between
  collection-per-tenant vs payload-based isolation, tenant data must not leak between queries,
  tenant isolation is impacting search performance, or scaling a SaaS product with per-tenant
  vector search.
---

# Multi-Tenancy with Qdrant

There are two isolation models: one collection per tenant, or all tenants in one collection separated by payload filters. The right choice depends on tenant count, data volume per tenant, and how strictly isolation must be enforced.

Wrong choice at design time is expensive to reverse. Choose based on tenant count and data volume before building.

## Choosing an isolation model

Use when: deciding the architecture before building.

- Use **collection-per-tenant** when: tenant count is under 500, per-tenant data volume varies widely, tenants need different vector configurations (dimensions, distance metrics), or regulatory requirements mandate physical isolation
- Use **payload-based isolation** when: tenant count is in the thousands or growing unbounded, average data volume per tenant is small (<100k vectors), and all tenants use the same embedding model — [qdrant.tech/documentation/guides/multiple-partitions](https://qdrant.tech/documentation/guides/multiple-partitions/)
- Collection-per-tenant has no cross-tenant bleed risk but increases operational overhead; payload-based isolation is operationally simpler but requires strict filter discipline in every query

## Payload-based isolation

Use when: using a shared collection with tenant ID filters.

- Store `tenant_id` as a keyword payload field on every point at index time; never index a point without it — [qdrant.tech/documentation/concepts/payload](https://qdrant.tech/documentation/concepts/payload/)
- Create a keyword index on `tenant_id` before ingesting data; without the index, every filtered query scans all points — [qdrant.tech/documentation/concepts/indexing](https://qdrant.tech/documentation/concepts/indexing/)
- Apply `tenant_id` as a `must` filter condition in every query; validate this at the application layer, not just the client layer
- Use named vectors with a tenant-specific quantization config if tenant data distributions differ significantly — [qdrant.tech/documentation/concepts/vectors](https://qdrant.tech/documentation/concepts/vectors/)

## Collection-per-tenant

Use when: managing dedicated collections per tenant.

- Name collections with a consistent convention: `tenant_{tenant_id}_documents`, `tenant_{tenant_id}_memory`; this enables glob-pattern operations and monitoring
- Create collections lazily on first tenant write, not eagerly on tenant signup; unused empty collections waste memory
- Cache the collection list in your application layer; calling `GET /collections` per request adds measurable latency at scale
- Use Qdrant's collection aliases to support zero-downtime tenant data rebuilds — [qdrant.tech/documentation/concepts/collections](https://qdrant.tech/documentation/concepts/collections/)

## Preventing cross-tenant data leaks

Use when: ensuring one tenant cannot access another's data.

- In payload-based isolation, enforce `tenant_id` filtering at the service layer with a middleware or query wrapper; do not trust client-supplied filters alone
- Write integration tests that explicitly verify a query from tenant A does not return results from tenant B
- For collection-per-tenant, validate collection names against a tenant registry before executing queries; a client that constructs collection names dynamically could access any collection
- Log all queries with `tenant_id` for audit; unexpected cross-tenant pattern in logs usually means a missing filter

## Scaling payload-based isolation

Use when: a shared collection is growing beyond 10M points and filter performance is degrading.

- Shard on `tenant_id` using Qdrant's custom sharding to co-locate each tenant's data on specific nodes — [qdrant.tech/documentation/guides/distributed-deployment](https://qdrant.tech/documentation/guides/distributed-deployment/)
- With custom sharding, filtered queries for a single tenant only scan one shard instead of the full collection
- Reassign shard ownership when tenant data volumes diverge significantly; a few large tenants on one shard will slow all tenants on that shard

## What NOT to Do

- Do not build a multi-tenant system with payload-based isolation without indexing the `tenant_id` field; it will scan every point on every filtered query
- Do not create collections eagerly for every new tenant; thousands of empty collections degrade Qdrant's metadata operations
- Do not share HNSW index tuning parameters across a collection where tenant data distributions are wildly different; one large tenant's data will dominate the graph structure
- Do not use collection-per-tenant for SaaS with unbounded tenant growth; operational overhead and metadata costs become unmanageable past a few thousand collections
