---
name: catalog-audit
description: Audit a UnoPim product catalog for data-quality gaps such as empty required attributes, missing descriptions, disabled products and uncategorized items. Use when the user asks to audit, review, health-check or find problems in their UnoPim catalog or a subset of it.
---

# Catalog audit

Produce a read-only data-quality report for the user's UnoPim catalog. This skill never writes data.

## Steps

1. Learn the catalog's shape before reading products:
   - `get_catalog_schema` returns the filter fields `search_products` accepts, including the value path of each filterable attribute, such as `values->common->color` or `values->channel_locale_specific->{channel}->{locale}->name`.
   - `search_settings` with `type` set to `channels`, then to `locales`, returns the channels and locales that exist.
   - `search_attributes` returns each attribute's `is_required`, `value_per_locale` and `value_per_channel`.
   - `search_families` returns the attribute families.
   Use only the codes these tools return; do not assume attribute codes.
2. Agree the scope with the user if they did not give one: a family, a status, or a list of SKUs. Default to enabled products.
3. Page through `search_products` with the agreed filters, at most 100 products per call, passing the returned `cursor` until it is empty or the scope is covered. For a large catalog, audit the first few pages and say that the report is a sample.
4. For products that need a closer look, call `get_product` with the SKU.
5. Check each product for:
   - required attributes with no value, in each channel and locale the attribute varies by
   - empty or very short name and description values
   - products with no category
   - disabled products the user may expect to be live
   - duplicate or near-duplicate names across different SKUs
6. Report the findings as a table with SKU, problem, channel or locale, and a suggested fix, followed by a short summary with counts per problem type.

## Rules

- Only call search and get tools in this skill.
- If a tool returns a permission error, tell the user which UnoPim permission their role is missing instead of retrying.
- Offer the `product-enrichment` skill to fix content gaps, and let the user decide.
