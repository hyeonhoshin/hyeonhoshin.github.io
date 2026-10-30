# hyeonhoshin.github.io

Personal academic page of Hyeonho Shin (신현호), built with Jekyll and served by GitHub Pages
(classic build, `github-pages` gem). No theme and no JavaScript. The only third-party request is
the Google Calendar embed on `calendar.html`; every other page loads nothing from other sites.

## Updating content

Almost every change is an edit to **`_data/data.yml`**. The comment block at the top of that file
explains the keys, which ones are required, and which section each one feeds.

- **News**: add a new item at the top of `news` (quote the date: `"2026.11"`). Only the first
  `news_visible` items (a plain number) are shown; older ones move into "More news" automatically.
- **Publications**: copy an existing entry. Put the owner in `**…**`, give the bare DOI
  (`10.1145/…`), and leave `pdf`, `video` or `award` empty to hide them.
- **Patents, service, education, honors, earlier**: rows with `what`, optional `sub`, `when`.
- The JSON-LD, `/llms.txt` and the page itself all read the same data, so they stay in sync.

### Adding, hiding or reordering a section

- The dated-row sections (Patents, Service, Education, Honors) are listed in `row_sections`
  in `data.yml`, in page order. Each entry names its list (`key`) and heading (`title`).
- To hide a section, delete its `row_sections` entry or empty its list (`honors: []`); the
  heading and its link in the top menu disappear with it. News and Publications also disappear
  when their lists are empty.
- To add a section such as "Talks", add a `talks:` list of `what`/`sub`/`when` rows and a
  `row_sections` entry `{ key: talks, title: Talks }`. The key becomes the section id, so keep
  it a plain word (see the comment above `row_sections` for words to avoid).

### Adding a page

Create `name.html` (or `.md`) in the repository root:

Use `layout: page`. `layout: default` is the home page only: it builds the whole page from
`data.yml` and ignores the page body.

```yaml
---
layout: page          # header with a link back to the home page, same footer and styles
title: Talks          # page heading and <title>
description: One sentence for search results.
# sitemap: false      # leave the page out of sitemap.xml
# noindex: true       # ask search engines not to index it
---
```

To link it from every page's footer, add `{ label: Talks, url: /name.html }` to `misc` in
`data.yml`. The footer shows the current page as plain text instead of a link.

`footer.updated` in `data.yml` sets the "Last updated" line; leave it empty to print the month
of the last build.

### Other files

| File | Purpose |
|---|---|
| `_config.yml` | Site title, description, URL, SEO and sitemap settings, excluded files |
| `_layouts/default.html` | Home page only (ignores page content) |
| `_layouts/page.html` | Sub-pages (`drawing.html`, `calendar.html`, `404.html`) |
| `_includes/site.css` | All styles; published as `/assets/site.css` through `assets/site.css` |
| `_includes/jsonld.html` | schema.org ProfilePage, Person and ScholarlyArticle data (end of the home page) |
| `_includes/rows.html`, `news-item.html`, `rich.html` | Dated rows, news items and `**bold**` handling |
| `_includes/foot.html`, `head.html` | Footer and `<head>` shared by all pages |
| `robots.txt`, `llms.txt`, `404.html` | Crawler, AI-reader and not-found pages |
| `favicon.svg`, `favicon.ico`, `apple-touch-icon.png` | Site icons (the PNG/ICO copies are for older browsers and iOS) |
| `assets/images/` | `profile-512.png` (share image) and the avatars made from it |
| `assets/fonts/pretendard/` | Pretendard 1.3.9 web font, self-hosted (see below) |

### Fonts

All text uses [Pretendard](https://github.com/orioncactus/pretendard) (Korean and Latin in one
typeface). The files in `assets/fonts/pretendard/` are copied unchanged from the npm package
`pretendard@1.3.9` (`dist/web/variable/pretendardvariable-dynamic-subset.css` and
`woff2-dynamic-subset/`), so the site loads no fonts from other servers. The font is split into
92 small files by character range, and a browser downloads only the ones a page needs (about
160 KB for the home page, 110 KB for the other pages). To update:

1. Replace `pretendardvariable-dynamic-subset.css`, `woff2-dynamic-subset/` and `LICENSE.txt`
   with `dist/web/variable/pretendardvariable-dynamic-subset.css`,
   `dist/web/variable/woff2-dynamic-subset/` and `dist/LICENSE.txt` from the new npm package.
2. Find the file that holds Basic Latin and put its name in the preload link in
   `_includes/head.html`:
   `grep -B1 'unicode-range: U+20-' assets/fonts/pretendard/pretendardvariable-dynamic-subset.css | grep -o 'PretendardVariable\.subset\.[0-9]*\.woff2'`
3. Update the version number here and in the table above.

### Changing the photo

Replace `assets/images/profile-512.png` (square, 512×512), then regenerate the avatars:

```sh
python3 - <<'PY'
from PIL import Image
src = Image.open('assets/images/profile-512.png').convert('RGBA')
src.save('assets/images/profile-512.png', optimize=True)   # drops editor metadata
for s in (192, 384):
    src.resize((s, s), Image.LANCZOS).save(f'assets/images/avatar-{s}.webp', quality=82, method=6)
src.resize((192, 192), Image.LANCZOS).save('assets/images/avatar-192.png', optimize=True)
PY
```

Keep the new photo small (a few hundred KB at most): it is the share image for every page.

## Building locally

Use a build that matches GitHub Pages (`github-pages` gem 232, Jekyll 3.10). Jekyll loads the
GitHub Pages plugins and settings only when it is started from the folder that holds the
`Gemfile`; the local helper changes into its own folder before building:

```sh
"$HOME/cv-build/build.sh" "$PWD" /tmp/cv-site   # never build into the repository
```

Without the helper, any folder with a `Gemfile` containing
`gem "github-pages", "232", group: :jekyll_plugins` works:
`bundle exec jekyll build --source "$repo" --destination /tmp/cv-site`, run from that folder.

Preview it over HTTP, not by opening the file: links such as `/assets/site.css` start at the site
root, so a page opened as `file://` shows no styles or images.

```sh
python3 -m http.server --directory /tmp/cv-site 8000   # then open http://localhost:8000
```

Check that `/tmp/cv-site/sitemap.xml` lists only the pages you meant to publish.

## Before committing

- A local pre-commit hook (not part of the repository) scans staged changes for personal data and
  stops commits that add new images or documents until they have been checked. Commits made in the
  GitHub web editor skip this hook, so prefer committing locally.
- Every file in the repository is public, even when the site does not show it. Keep working
  notes and drafts outside the repository (or in `_private/`, which is git-ignored). Jekyll also
  turns any `.md` file in the root into a page, so do not leave notes there.
- Publish only public, verified information: no phone numbers, birth dates, student IDs or scanned
  certificates, and nothing about work that is still under review.

## License

`LICENSE.txt` is the MIT license of the original online-cv theme this repository was forked from.
The theme code has since been replaced, and the license does not apply to the site's content.
Text, photos and drawings: © Hyeonho Shin, all rights reserved.
The Pretendard font is © Kil Hyung-jin and licensed under the SIL Open Font License 1.1
(`assets/fonts/pretendard/LICENSE.txt`).
