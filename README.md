# frame-embed

A static, full-window wrapper that shows one fixed URL in an iframe.
No build step, no dependencies — just `index.html`.

## 1. Set the URL

Edit the `src` of the `<iframe>` in `index.html` (it currently has a placeholder):

```html
src="https://YOUR-SITE.vercel.app/properties/eon-city"
```

The URL is hard-coded on purpose. There is no `?url=` parameter, so this page
cannot be used to frame arbitrary sites under your domain.

## 2. Put it in its own git repo

This folder is self-contained. Move it **out** of the `realestate` API project first
(so it is not a nested repo), then:

```bash
cd frame-embed
git init
git add .
git commit -m "Initial embed"
git branch -M main
git remote add origin <your-new-repo-url>
git push -u origin main
```

## 3. Deploy on Vercel

1. Vercel → **Add New → Project** → import the repo.
2. Framework Preset: **Other**. Leave Build Command and Output Directory empty.
   Root Directory: the repo root.
3. Deploy. You get a stable `https://<project>.vercel.app` URL (add a custom domain
   in Project → Settings → Domains if you want).

Changing the embedded URL later = edit `index.html`, commit, push. Vercel redeploys.

## 4. Let the embedded site be framed

The site inside the iframe must allow your new Vercel URL, or the browser shows a
blank/refused frame. On the embedded site (e.g. Next.js `next.config.js`):

```js
async headers() {
  return [{
    source: "/:path*",
    headers: [{
      key: "Content-Security-Policy",
      value: "frame-ancestors 'self' https://<project>.vercel.app https://your-custom-domain.com",
    }],
  }];
},
```

Remove any `X-Frame-Options` header on that site — it overrides `frame-ancestors` in some browsers.

## Limits of iframes

- Cookies/localStorage inside a cross-site iframe are partitioned, so logins may not persist.
- The address bar shows this wrapper's URL, not the page inside the iframe.
- Custom request headers (like `x-tenant-domain`) cannot be sent by an iframe; the embedded
  site must work out its tenant from its own hostname.
