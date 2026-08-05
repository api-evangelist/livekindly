---
name: Enumerate LIVEKINDLY brands, partners and open roles
description: >-
  Read the LIVEKINDLY Collective brand directory, partner directory and careers listings from the
  WordPress REST content API — and understand exactly how thin those payloads are before relying
  on them.
api: openapi/livekindly-content-openapi.yml
operations: [getBrand, getBrandById, getPartner, getPartnerById, getJob, getJobById, getMedia, getSearch]
generated: '2026-08-04'
method: generated
source: >-
  Derived from openapi/livekindly-content-openapi.yml and data-model/livekindly-data-model.yml,
  with item counts and field lists taken from live anonymous GETs on 2026-08-04. Every operationId
  is verified present in the spec.
---

# Enumerate LIVEKINDLY brands, partners and open roles

Three custom post types on `https://thelivekindlyco.com/wp-json/wp/v2` carry the business-facing
directories: `brand`, `partner` and `job`. All three are anonymously readable.

## Authentication

None.

## Steps

1. **List the brands.** Call `getBrand`.

   ```
   GET https://thelivekindlyco.com/wp-json/wp/v2/brand?per_page=100&_fields=id,slug,title,link
   ```

   Four are published: Fry's Family Food Co. (`frys-family-food-co`), Like Meat (`like-meat`),
   Oumph! (`oumph`), The No Meat Company (`the-no-meat-company`).

2. **List the partners.** Call `getPartner` the same way. Four are published: PHW Group
   (`phw-group`), RCL Foods (`rcl-foods`), Ospelt (`ospelt`), Coest (`coest`).

3. **List open roles.** Call `getJob`. Six are published at time of profiling.

4. **Expect nothing more than a name.** All three types are registered without `content`,
   `excerpt`, `featured_media` or any taxonomy. A response gives you `id`, `date`, `slug`,
   `status`, `type`, `link`, `title`, `template`, `class_list`, an **empty** `acf` array, and
   Yoast's `yoast_head` / `yoast_head_json`.

5. **Get the substance from `yoast_head_json` or the page.** If you need a description, the
   Schema.org block inside `yoast_head_json` (`og:description`, `description`) is the only
   structured prose in the payload. Anything richer requires fetching the HTML page at `link`.

6. **Find related coverage.** There is **no** relationship field linking a brand to the posts about
   it. Use `getSearch` (`GET /wp/v2/search?search=Oumph`) or `getPosts?search=...` and join on the
   name yourself. Brand and partner ids never appear in a post payload.

7. **Find brand imagery.** Each item exposes a `wp:attachment` link relation. Follow it, or query
   `getMedia` with `parent` set to the brand id. The media library (1,145 items) is where the real
   depth on this host lives — `alt_text`, `caption` and the full `media_details` size set are
   populated.

## Rules

- **Do not infer a brand↔partner relationship.** The API models none. The company narrative
  connects manufacturers to brands; the JSON does not, and asserting the link from these payloads
  would be a guess.
- **Ids are bare integers with no type prefix.** Id `447` is only meaningful alongside the
  collection name. Prefer `slug` as your join key.
- **Titles are HTML-entity encoded** (`Fry&#8217;s Family Food Co.`). Decode before display.
- **Job listings are not a job feed.** No location, department, employment type, salary or closing
  date is exposed — only a title, a slug and a permalink. Do not synthesize those fields. Note also
  that the job sitemap has not been regenerated since 2022-01-17 even though six roles are live in
  the API.
- Counts above were observed on 2026-08-04 via `X-WP-Total`. Re-read that header rather than
  trusting them.
