---
name: catalog-structure
description: Plan and set up UnoPim catalog structure such as attributes, attribute options, attribute groups, attribute families and categories for a new product line. Use when the user asks to model a product type, add attributes, options or families, or organize categories in UnoPim.
---

# Catalog structure

Design the attributes, groups, families and categories a product line needs, and create them in UnoPim after the user approves the plan.

## Steps

1. Call `search_attributes`, `search_attribute_groups`, `search_families` and `search_categories` to see what already exists. Reuse existing attributes and groups wherever they fit instead of creating near-duplicates.
2. For select, multiselect and checkbox attributes, call `search_attribute_options` to see their existing options.
3. Propose a plan as a table: each new or changed attribute with its code, type, options, whether it is required, and whether it varies by locale or channel; each attribute group; each family with the groups and attributes it gets; and each category with its parent.
4. After the user approves, create items in dependency order, in batches of at most 50:
   1. Attributes with `upsert_attributes`. Send options in the attribute's `options` list; options are matched by code, and options left out are kept.
   2. Attribute groups with `upsert_attribute_groups`.
   3. Families with `upsert_families`, passing `attribute_groups` as a list of group codes with their attribute codes. This only adds to a family; it never removes groups or attributes.
   4. Categories with `upsert_categories`, parents before children.
5. Report what was created or updated, with any errors the tools returned.

## Rules

- Never call an upsert tool before the user has approved the plan in this conversation.
- Use codes made of letters, digits and underscores, and never change the code of an existing item.
- An attribute's type cannot change after it is created, so confirm the type before creating it.
- If a tool returns a permission error, tell the user which UnoPim permission their role is missing instead of retrying.
