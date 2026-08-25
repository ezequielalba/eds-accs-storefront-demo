# Promo Banner Block

## Overview

The Promo Banner block displays a curated grid of products from a given category, intended for use as a marketing/promotional module on pages (e.g. homepage or landing pages). It fetches products directly via a `productSearch` GraphQL query filtered by category and renders them as a heading plus a responsive grid of product cards (image, name, price).

## Configuration Options

Block configuration is read via `readBlockConfig(block)`.

| Option | Effect |
|--------|--------|
| `category-id` | The category ID used to filter products via the `categoryIds` attribute. Defaults to `''` (empty), which will return no meaningful results unless a valid ID is authored. |
| `heading` | Heading text displayed above the product grid. Defaults to `'Featured Products'`. |
| `max-products` | Maximum number of products to fetch and display. Defaults to `4` if not set or not a valid number. |

## Integration

<!-- ### URL Parameters

No URL parameters are read or written by this block. -->

### GraphQL

The block queries commerce data directly via `CS_FETCH_GRAPHQL.fetchGraphQl` (from `scripts/commerce.js`), independent of any dropin:

- `productSearch(phrase: "", filter: [{ attribute: "categoryIds", eq: $categoryId }], page_size: $pageSize)` — returns `productView` items with `name`, `sku`, `urlKey`, `images`, and `price` (final/regular amounts).

Product links are built with `getProductLink(urlKey, sku)` from `scripts/commerce.js`.

<!-- ### Events

This block does not listen for or emit any events. -->

<!-- ### Local Storage

This block does not use localStorage. -->

## Behavior Patterns

### Rendering Flow

1. **Initialization**: Block renders a heading and a `Loading products...` placeholder immediately.
2. **Fetch**: `fetchCategoryProducts` requests up to `max-products` items in `category-id` via GraphQL.
3. **Render**: On success, each product is rendered as a linked card with image (when available), name, and final price (when available). If no products are returned, an empty-state message (`No products found.`) is shown instead.

### Error Handling

- **Fetch errors**: The GraphQL request is wrapped in `try/catch`; errors are logged with `console.error('Promo banner: failed to fetch products', error)` and the products container falls back to an `Unable to load products.` message.
- **Missing data**: Rendering guards against missing `image` and `price` fields, omitting the `<img>` or price `<span>` when not present.
