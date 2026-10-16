---
name: product-enrichment
description: Write or improve UnoPim product content such as names, descriptions, short descriptions and SEO fields for one product or a batch of SKUs, then save it back after the user approves. Use when the user asks to enrich, write, rewrite, translate or fill in product content in UnoPim.
---

# Product enrichment

Draft better product content and save it to UnoPim only after the user approves the exact changes.

## Steps

1. Call `search_attributes` to find the attribute codes to fill and whether each one varies by locale or channel, and `search_settings` with `type` set to `channels` and then `locales` to find the codes that exist. Use only codes these tools return.
2. Load each product with `get_product`, or find products with `search_products` when the user describes a set instead of listing SKUs. Call `get_catalog_schema` first if you need filter fields for the search.
3. Write each new value at the same place in `values` where the product already stores that attribute, for example `common`, `locale_specific`, `channel_specific` or `channel_locale_specific`, matching the attribute's locale and channel settings.
4. Draft the new values from the product's existing data. Do not invent specifications, certifications, materials, dimensions or claims that the product data does not support; ask the user when a fact is missing.
5. Show the user a before and after view for every product and attribute you plan to change.
6. After the user approves, save with `upsert_products`, sending only the SKU and the values that changed, in batches of at most 50 products.
7. Report which SKUs were saved and list any errors the tool returned.

## Rules

- Never call `upsert_products` before the user has approved the changes in this conversation.
- Never change `type`, `attribute_family_id`, `status`, `parent_sku` or `super_attributes` in this skill.
- For translations, write natural text in the target locale and keep units, codes and brand names unchanged.
- If a tool returns a permission error, tell the user which UnoPim permission their role is missing instead of retrying.
