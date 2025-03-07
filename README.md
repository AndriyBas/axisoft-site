# webflow-fetch

Fetch Webflow site content, purge (remove unused) and inline CSS, fetch 'script.js', 'jquery.js', build 'sitemap.xml', 'robots.txt', _redirects

Prepare to be deployed to Cloudflare Pages.

## Required params:
`site`: from where to fetch the Webflow site content. Your ".webflow.io" version.

`targetHost`: what will be deployment desitnation. Will use it for relative links, sitemap, etc. Recommend to use 'https://www.company.com' format (with 'www').

> TODO: if you need pagination — add your targetHost to `CORSANYWHERE_WHITELIST`

## Publishing details
The Workflow and “Build” takes only what is on the current Dev version of the Webflow site, and pushes it to Live. So to have the changes on Live — you need to `ALWAYS` publish them to Dev first (including new CMS entries). Just “Publish” / “Unpublish” for CMS entries without the site publishing will not work.

## Pagination / Search details
> ☝️ Important: default Webflow pagination does not work (cos of dynamic query params). 

Will work only with [CMS Load](https://www.finsweet.com/attributes/cms-load) / [CMS Filter](https://www.finsweet.com/attributes/cms-filter) Finsweet libs. Don't forget to add the "targetHost" to the `CORSANYWHERE_WHITELIST` .ENV on the Proxy Server.

## Image processing and re-deploy
The script will find all `<img data-sb-process-img />` tags (note the attribute `data-sb-process-img`), fetch all resource links inside (including `srcset`), re-deploy them to `assets/` folder (without optimization, as is), and update the links on that page (not on other pages). Use it for images that need to be optimized (the ones that impact FCP / LCP scores)

## When 'sitemap.xml' not found
- The script will generate own sitemap based on the links from the Home page (to fetch all pages — add them as <a> on the Home page. Can be 'display:none'.)
- To NOT add link to the sitemap (when building custom) - use attribute 'data-sb-skip-sitemap' for <a>
- To NOT fetch page - use attribute 'data-sb-skip-fetch' for <a>

## For 'robots.txt'
- Will take `robots` (multiline string) from the config (if available).
- If no 'robots' in the config, will take '{site}/robots.txt' (if available).

## For redirects (301/302)
Add `redirects` (multiline string) in the config. One per line. In the format "{old} {new} {redirect_code}" ([redirect docs](https://developers.cloudflare.com/pages/platform/redirects/)). For example:
```
/old/path /new/destination 301
/old/path2 /new/destination2 302
```

## Custom headers
Add `headers` (multiline string) in the config. Default example disables robots indexing of '*.pages.dev' (NOTE: 2 spaces before the header). See [header docs](https://developers.cloudflare.com/pages/platform/headers/). 
```
https://:project.pages.dev/*
  X-Robots-Tag: noindex
```

## Static file hosting
All `content/` folder is erased during each re-build, and new pages are re-fetched, processed, and saved in `content/` folder. Excpet for the `content/sb_static` folder. To host static files/assets that would persist between builds (and won't be erased) — add them to `content/sb_static`folder (commit and push to 'main')

## Embed external scripts
`<script></script>` tags with `data-sb-embed-script` tag will be fetched (by "src" link) and embeded directly (NOTE: as sync scripts, NOT async). Resulting embeded script will have `data-sb-result-script` tag.