# Enriched Product Block

## Overview

The Enriched Product block displays a single product as a card (image, name, SKU, price) augmented with third-party enrichment data — a sustainability score badge and an estimated delivery date — sourced from a custom resolver on a dedicated API Mesh instance. Unlike other commerce blocks in this project, it talks to its mesh endpoint directly via `fetch` rather than through the shared `scripts/commerce.js` GraphQL clients.

## Configuration Options

Block configuration is read via `readBlockConfig(block)`.

| Option | Effect |
|--------|--------|
| `sku` | The SKU of the product to fetch and display. **Required** — if missing, the block renders `No SKU configured for this block.` and does not attempt a fetch. |

## Integration

<!-- ### URL Parameters

No URL parameters are read or written by this block. -->

### GraphQL

The block queries a hardcoded API Mesh endpoint (`MESH_ENDPOINT` in `enriched-product.js`) directly via `fetch`, independent of the site's shared `CORE_FETCH_GRAPHQL`/`CS_FETCH_GRAPHQL` clients and their configured `commerce-endpoint`. A single query requests two root fields for the given `$sku`:

- `products(skus: [$sku])` — catalog data: `sku`, `name`, `__typename`, `images(roles: [])` (unfiltered, first image is used), and, depending on `__typename`, either `price.final.amount` (`SimpleProductView`) or `priceRange.minimum.final.amount` (`ComplexProductView`).
- `Enrichment_getProductEnrichment(sku: $sku)` — a custom mesh resolver returning `sku`, `name`, `sustainabilityScore`, `estimatedDelivery`, and `enrichedAt`.

<!-- ### Events

This block does not listen for or emit any events. -->

<!-- ### Local Storage

This block does not use localStorage. -->

## Behavior Patterns

### Rendering Flow

1. **Configuration check**: If no `sku` is configured, renders `No SKU configured for this block.` and stops.
2. **Loading state**: Renders `Loading product details...` while the request is in flight.
3. **Fetch**: Sends the combined catalog + enrichment query to the mesh endpoint for the configured SKU.
4. **Render**: On success, renders a card with (when available) the first product image, name, SKU, and formatted price. If enrichment data is present, appends a sustainability badge (`Excellent` ≥ 80, `Good` ≥ 60, otherwise `Fair`), the estimated delivery text, and an `Enriched at` timestamp (only shown if `enrichedAt` is present).

### Error Handling

- **Missing SKU**: Skips the fetch entirely and shows a configuration message instead of an error.
- **HTTP errors**: A non-OK response throws and is caught, logging `Enriched product block failed:` to the console and rendering `Unable to load product data.`.
- **GraphQL errors**: Errors in the response payload are logged via `console.error('Mesh GraphQL errors:', ...)`, then thrown and handled the same way as HTTP errors.
- **Product not found**: If the `products` query returns no rows, renders `Product not found.` instead of attempting to build a card.
- **Missing fields**: Rendering guards against a missing image, price, or enrichment payload, omitting the corresponding markup rather than failing.
