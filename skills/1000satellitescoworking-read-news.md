---
name: Read the 1000 Satellites news feed
description: Retrieve and summarise company news from 1000 Satellites — new location openings, funding and
  company announcements — from the anonymously readable WordPress content API on 1000satellites.de.
api: openapi/1000satellitescoworking-content-api-openapi.yml
operations: [listPosts, getPost, listCategories, listTags, getMediaItem]
---

# Read the 1000 Satellites news feed

1000 Satellites publishes company news as WordPress posts. The feed is anonymously readable — no key, no
token, no sign-up. There were 28 posts as of 2026-09-05.

Base URL: `https://1000satellites.de/wp-json`

## 1. Authenticate

Do not. This surface takes no credential. If you receive `401 rest_forbidden` you have hit an
administrative route (`/wp/v2/users`, `/wp/v2/settings`) rather than a content route — check the path.

## 2. List posts — `listPosts`

```
GET /wp/v2/posts?per_page=20&page=1&orderby=date&order=desc&wpml_language=en
```

- `per_page` maxes out at 100; the default is 10.
- Read `X-WP-Total` and `X-WP-TotalPages` from the response headers to know how far to paginate. Do not
  page blindly until you get an empty array.
- `wpml_language` accepts `en` or `de`. **German is the default** — always set `en` explicitly if you want
  English, including on error messages.
- Narrow by date with `after` / `before` (ISO 8601), or by text with `search`.
- Trim the payload with `_fields=id,date,slug,link,title,excerpt` — post `content.rendered` is Elementor
  page-builder HTML and is very large. Fetching 20 full posts without `_fields` will return hundreds of
  kilobytes of markup for very little signal.

## 3. Fetch one post — `getPost`

```
GET /wp/v2/posts/{id}?_embed&wpml_language=en
```

`_embed` inlines the featured image and terms so you do not need follow-up calls to `getMediaItem`,
`listCategories` or `listTags`.

## 4. Classify — `listCategories` and `listTags`

```
GET /wp/v2/categories?per_page=100
GET /wp/v2/tags?per_page=100
```

Post objects carry `categories[]` and `tags[]` as ID arrays; resolve them against these collections, or
just use `_embed`.

## Rules

- **Titles and excerpts arrive as HTML**, under `title.rendered` and `excerpt.rendered`, with HTML entities
  encoded. Decode before quoting.
- **Match errors on `code`, never on `message`.** The message text is localised and this host serves German
  by default: `{"code":"rest_forbidden","message":"Du bist leider nicht berechtigt…","data":{"status":401}}`.
- **Nothing here is rate-limit signalled.** No `RateLimit-*` or `Retry-After` header is ever returned, but
  the site runs Varnish and Wordfence, so throttling exists and is invisible until it blocks you. Keep
  concurrency low and pace politely — treat a sudden 403 or connection reset as a soft limit.
- **This surface is read-only for you.** Every operation in the linked spec is a GET. Do not attempt POST,
  PUT or DELETE: they require WordPress credentials, they are not part of this contract, and there is no
  idempotency or reversal mechanism if one succeeded.
