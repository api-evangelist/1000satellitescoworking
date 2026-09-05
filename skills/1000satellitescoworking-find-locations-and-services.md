---
name: Answer questions about 1000 Satellites locations and services
description: Use site search and the WordPress pages collection on 1000satellites.de to answer questions
  about 1000 Satellites' coworking locations, meeting rooms, membership pricing and terms — and to know
  when the answer is not available from any API.
api: openapi/1000satellitescoworking-content-api-openapi.yml
operations: [searchSite, listPages, getPage, listMedia, listTypes, listTaxonomies]
---

# Answer questions about 1000 Satellites locations and services

Read this first: **there is no locations API, no availability API and no booking API.** Location and
meeting-room information exists only as editorial web pages. The plugin that holds structured listing data
(Listdom) advertises a public feed at `/wp-json/listdom/v1/public/listings` but it returns
`403 {"success":0,"message":"The public listings feed is unavailable."}`. Do not present page prose as
structured availability, and never state that a room is free — you cannot know that.

Base URL: `https://1000satellites.de/wp-json`

## 1. Search the site — `searchSite`

```
GET /wp/v2/search?search=Mannheim&per_page=20&type=post&wpml_language=en
```

Returns lightweight `{id, title, url, type, subtype}` hits. This is the cheapest entry point — start here
rather than pulling the whole pages collection.

## 2. Pull the page — `getPage`

```
GET /wp/v2/pages/{id}?_fields=id,link,title,content&wpml_language=en
```

Location pages live under `/standorte/<region>/<satellit>/` (German) and `/en/locations/…` (English).
Meeting rooms are separate pages beneath a location.

## 3. Enumerate pages — `listPages`

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent&wpml_language=en
```

Use `parent` to reconstruct the location → meeting-room hierarchy. Paginate on `X-WP-TotalPages`.

## 4. Images — `listMedia`

```
GET /wp/v2/media?parent={page_id}&_fields=id,source_url,alt_text,media_details
```

## Rules

- **`content.rendered` is Elementor markup**, not clean prose. Strip tags and expect layout div soup and
  inline styles. Budget for it, or prefer `excerpt.rendered` where present.
- **Pricing is not in this API.** The published price list is a human page at
  `https://1000satellites.de/preise` (ten membership tiers, €39–€3,399/month, ex VAT, one-month minimum
  term and 14 days' notice). It is captured in `plans/1000satellitescoworking-plans-pricing.yml`. Quote it
  from there and cite the page; do not synthesise a price.
- **Booking is a human flow.** `https://1000satellites.de/buchung/` is a web form, and meeting rooms,
  events and business-address services are quoted on request. If a user asks you to book, hand them the URL
  or the contact details (`info@1000satellites.com`, +49 621 39999 541) — there is no API to call, and
  therefore no way to cancel or reverse anything you did.
- **Answer in the language asked.** `wpml_language=en|de`. German is the default; English pages are a
  parallel `/en/` tree, and not every German page has an English twin.
- **Match errors on `code`, not `message`** — message text is German by default.
