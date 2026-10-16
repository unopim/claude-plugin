---
name: catalog-audit
description: Audit a UnoPim product catalog for data-quality gaps such as empty required attributes, missing descriptions, disabled products and uncategorized items. Use when the user asks to audit, review, health-check or find problems in their UnoPim catalog or a subset of it.
---

# Catalog audit

Produce a read-only data-quality report for the user's UnoPim catalog. This skill never writes data.

## Steps

1. Learn the catalog's shape before reading products:
   - `get_catalog_schema` returns the filter fields `search_products` accepts, including the value path of each filterable attribute, such as `values->common->color` or `values->channel_locale_specific->{channel}->{locale}->name`.
   - `search_settings` with `type` set to `channels` returns the channels. With `type` set to `locales` it returns every locale UnoPim ships, enabled or not, so pass the filter `[{"field": "status", "operator": "=", "value": 1}]` to get the active ones.
   - `search_attributes` returns each attribute's `is_required`, `value_per_locale` and `value_per_channel`.
   - `search_families` returns the attribute families.
   Use only the codes these tools return; do not assume attribute codes.
2. Agree the scope with the user if they did not give one: a family, a status, or a list of SKUs. Default to enabled products.
3. Page through `search_products` with the agreed filters, at most 100 products per call, passing the returned `next_cursor` as `cursor` until `has_more` is false or the scope is covered. Each result holds only `id`, `sku`, `type`, `status` and `attribute_family_id`, so use this step for triage: status, family and SKU-level checks, and filters on attribute value paths to find products with empty values.
4. Content checks need `get_product`, one call per product. Load at most 50 products this way and say in the report that it is a sample of the scope. UnoPim limits each tool to about 60 calls per minute per user; if a tool returns `Rate limit exceeded`, stop calling it, tell the user, and continue after a minute instead of retrying in a loop.
5. Check each loaded product for:
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
