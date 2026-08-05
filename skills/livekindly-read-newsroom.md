---
name: Read the LIVEKINDLY newsroom
description: >-
  Retrieve and page through LIVEKINDLY Collective's corporate news releases and LiveKindly Blog
  articles from the anonymously readable WordPress REST content API, with correct pagination,
  sparse fieldsets and error handling.
api: openapi/livekindly-content-openapi.yml
operations: [getPosts, getPostsById, getCategories, getCategoriesById, getMediaById]
generated: '2026-08-04'
method: generated
source: >-
  Derived from openapi/livekindly-content-openapi.yml, conventions/livekindly-conventions.yml and
  errors/livekindly-problem-types.yml. Every operationId is verified present in the spec.
---

# Read the LIVEKINDLY newsroom

LIVEKINDLY Collective publishes no developer documentation. Its newsroom is nonetheless available
as JSON from the WordPress REST API at `https://thelivekindlyco.com/wp-json/wp/v2`. 39 posts are
published; the most recent is dated 2026-06-09.

## Authentication

None. Every operation in this skill is anonymous over HTTPS. Do not send credentials.

## Steps

1. **List posts, newest first.** Call `getPosts`.

   ```
   GET https://thelivekindlyco.com/wp-json/wp/v2/posts?per_page=20&orderby=date&order=desc&_fields=id,slug,link,date_gmt,title,excerpt,categories
   ```

   Use `_fields` on every call. Without it each post carries a large `yoast_head` HTML blob and a
   full rendered `content` body, which is wasted context if you only need headlines.

2. **Page correctly.** Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, or follow
   the RFC 8288 `Link` header's `rel="next"`. `per_page` is capped at 100 — asking for more returns
   HTTP 400 `rest_invalid_param`, not a clamped result.

3. **Filter by topic.** Call `getCategories` to resolve slugs to ids, then pass the id to `getPosts`
   as `categories`. Three categories exist: `livekindly-blog` (22 posts), `stories` (9),
   `uncategorized` (14).

4. **Filter by date window.** `getPosts` accepts `after` and `before` (ISO 8601), plus
   `modified_after` / `modified_before`. Note that pre-2026 releases carry a date-only value that
   serializes with a zeroed time component — order on `date_gmt`, not `date`, if precision matters.

5. **Fetch one post in full.** Call `getPostsById` with the id from step 1. `title.rendered`,
   `excerpt.rendered` and `content.rendered` are HTML strings with HTML entities (`Fry&#8217;s`);
   decode entities and strip markup before summarizing.

6. **Resolve the hero image.** A post's `featured_media` is a media id. Either call `getMediaById`,
   or add `_embed=wp:featuredmedia` to the step-1 call and read `_embedded`. `alt_text` and
   `caption` are populated on this site and are worth carrying through.

## Rules

- **No conditional requests.** No `ETag` or `Last-Modified` is emitted, and `cache-control` is
  `max-age=0`. Poll on a schedule and diff on `modified_gmt`; do not expect a 304.
- **No rate-limit headers.** Nothing tells you your budget. The site sits behind a Sucuri WAF, so
  be conservative — serialize requests and back off on any non-200.
- **Branch on content-type before parsing.** A WAF block returns HTTP 403 with an **HTML**
  interstitial, not the JSON error envelope. Parsing it as JSON will throw.
- **Error envelope** is `{"code": ..., "message": ..., "data": {"status": ...}}` — not RFC 9457.
  On 400, read `data.params` for the per-field reason.
- **Do not paginate the author.** `getUsers` is edge-blocked (403). If you need a byline, take it
  from `_embed=author` on the post where it is exposed.
- This is a CMS content API on a corporate marketing site. It carries no product, inventory,
  nutrition or pricing data — do not represent it as a LIVEKINDLY product API.
