# Server redirect instructions and testing

This branch contains non-destructive server redirect configs to fix Search Console indexing issues without changing frontend HTML/CSS.

Files added:
- _redirects         (Netlify)
- vercel.json         (Vercel)
- README_REDIRECTS.md (this file)

If your site is hosted on Netlify or Vercel, drop the corresponding file into the repo root and deploy. If you host elsewhere, see the examples below.

Apache (.htaccess)
-------------------
RewriteEngine On
# force https & www
RewriteCond %{HTTP_HOST} !^www\.russellbernabo\.com$ [NC,OR]
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://www.russellbernabo.com/$1 [L,R=301]

# specific 404 to canonical
Redirect 301 /directions.html /about/
Redirect 301 /about_frame.html /gilded-surfaces/
Redirect 301 /saddles.html /
Redirect 301 /index-2.html /

Nginx
-----
# redirect non-www to www
server {
  listen 80;
  server_name russellbernabo.com;
  return 301 https://www.russellbernabo.com$request_uri;
}
server {
  listen 443 ssl;
  server_name russellbernabo.com;
  # TLS config omitted
  return 301 https://www.russellbernabo.com$request_uri;
}

# specific file redirect example
location = /directions.html { return 301 https://www.russellbernabo.com/about/; }

Cloudflare
----------
Use a Page Rule: If host equals russellbernabo.com -> Forwarding URL (301) -> https://www.russellbernabo.com/$1

Validation (after deploy)
-------------------------
1) curl -I http://russellbernabo.com/  -> should return 301 -> https://www.russellbernabo.com/
2) curl -I http://russellbernabo.com/directions.html -> 301 -> https://www.russellbernabo.com/about/
3) curl https://www.russellbernabo.com/about/ | grep 'rel="canonical"' -> should show canonical (unchanged)
4) Re-submit https://www.russellbernabo.com/sitemap.xml in Google Search Console and Request Indexing for priority pages.

Notes
-----
- I reverted the previous HTML edits on this branch so frontend HTML/CSS match the production files exactly (no layout changes). The only files added/changed are server redirect configs and robots.txt was left as original.
