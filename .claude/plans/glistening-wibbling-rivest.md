# Plan: Category Pages, Pagination, and Pagefind Search

## Context

The blog has 4 posts across 2 categories with no way to browse by category, paginate listings, or search content. This plan implements the three features from the concept document, renames the existing categories (`update` → `progress`, `concept` → `engineering`), and makes the category system dynamic so new categories can be introduced without editing templates or nav.

**Key deviations from concept**:
- **Drop `_scripts/generate_pages.sh` and `_data/categories.yml`** — replace with `jekyll-paginate-v2` autopages, which auto-generates category index pages from post frontmatter. Zero scripts, zero data files.
- **Dynamic nav** — `site.categories` drives the nav links via Liquid iteration instead of hardcoded links. Adding a new category requires only using it in a post's frontmatter.
- **Open-ended categories in dev-blog skill** — the draft command presents existing categories as options but accepts new ones. The style profile's per-category voice sections are guidance, not gatekeeping.

---

## Phase 1 — Category Rename

Rename categories in all 4 existing posts:

| Post | Old | New |
|------|-----|-----|
| `_posts/2026-02-07-consolidating-the-kernel-monorepo.md` | `update` | `progress` |
| `_posts/2026-02-07-introducing-the-dev-blog.md` | `update` | `progress` |
| `_posts/2026-02-07-dev-blog-architecture.md` | `concept` | `engineering` |
| `_posts/2026-02-07-kernel-architecture-overview.md` | `concept` | `engineering` |

Each post: change `category: update` → `category: progress` or `category: concept` → `category: engineering` in frontmatter.

**Note**: This changes post URLs (e.g., `/update/2026/...` → `/progress/2026/...`). Acceptable since the blog is new with no inbound links.

---

## Phase 2 — Dependencies and Configuration

### `Gemfile` — add jekyll-paginate-v2

```ruby
group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-paginate-v2"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
end
```

Run `bundle install`.

### `_config.yml` — add plugin, pagination, autopages

```yaml
plugins:
  - jekyll-feed
  - jekyll-paginate-v2
  - jekyll-seo-tag
  - jekyll-sitemap

pagination:
  enabled: true
  per_page: 10
  sort_reverse: true
  sort_field: "date"

autopages:
  enabled: true
  categories:
    enabled: true
    layouts: ["category.html"]
    title: ":cat"
    permalink: "/categories/:cat/"
  tags:
    enabled: false
  collections:
    enabled: false
```

Also fix exclude list: `.playwright` → `.playwright-mcp`.

### Playwright output directory

All playwright-cli screenshots and recordings go to `_capture/.playwright` (not `.playwright-mcp/`). The `_capture/` directory is already excluded from Jekyll build. Update references in:
- `.claude/CLAUDE.md` — Playwright section
- `.claude/skills/dev-blog/commands/draft.md` — Step 8 screenshot path
- `.claude/skills/dev-blog/commands/publish.md` — Step 2 and Step 6 screenshot paths

---

## Phase 3 — Layouts and Includes

### New: `_layouts/category.html`

Inherits `default`. Uses `paginator.posts` from autopages. Reuses `.post-list` markup pattern from `home.html`. Conditionally includes pagination controls when `paginator.total_pages > 1`.

### New: `_includes/pagination.html`

Previous/next links using `paginator.previous_page_path` and `paginator.next_page_path`, piped through `| relative_url`.

### Update: `_includes/nav.html`

**Dynamic** category links from `site.categories`:

```html
<nav>
  <a href="{{ '/' | relative_url }}" class="nav-title">{{ site.title }}</a>
  <div class="nav-links">
    {% assign sorted_cats = site.categories | sort %}
    {% for cat in sorted_cats %}
    <a href="{{ '/categories/' | append: cat[0] | append: '/' | relative_url }}"
       {% if page.title == cat[0] %}class="active"{% endif %}>
      {{ cat[0] | capitalize }}
    </a>
    {% endfor %}
    <a href="{{ '/search/' | relative_url }}">Search</a>
  </div>
</nav>
```

New categories automatically appear in the nav when posts use them — no template edits needed.

### Update: `_layouts/post.html`

Add Pagefind data attributes to existing elements (no structural changes):
- `data-pagefind-body` on `<article>`
- `data-pagefind-meta="title"` on `<h1>`
- `data-pagefind-meta="date:..."` on `<time>`
- `data-pagefind-filter="category"` via hidden span (existing category badge only in `home.html`)
- `data-pagefind-filter="tag"` on each `.tag` span

---

## Phase 4 — Search Page

### New: `search.html`

Standalone page at `/search/` with Pagefind UI:

```html
<link href="{{ '/pagefind/pagefind-ui.css' | relative_url }}" rel="stylesheet">
<script src="{{ '/pagefind/pagefind-ui.js' | relative_url }}"></script>
<script>
  window.addEventListener('DOMContentLoaded', function () {
    new PagefindUI({
      element: "#search",
      baseUrl: "{{ site.baseurl }}/",
      showImages: false
    });
  });
</script>
```

`baseUrl: "/jaime/"` ensures search result links include the subpath prefix. `data-pagefind-ignore` on the search container prevents self-indexing.

### Keyboard shortcut: `/` to focus search

After Pagefind UI initializes, customize the placeholder and add a global `/` listener:

```js
var input = document.querySelector('#search input');
if (input) input.placeholder = 'Search (press / to focus)';

document.addEventListener('keydown', function (e) {
  if (e.key === '/' && !['INPUT', 'TEXTAREA', 'SELECT'].includes(document.activeElement.tagName)) {
    e.preventDefault();
    input.focus();
  }
});
```

This mirrors GitHub's `/` behavior — ignored when already typing in a form field.

---

## Phase 5 — Styling (invoke /frontend-design)

Use `/frontend-design` skill for CSS to maintain design quality.

### `assets/css/layout.css`

Update nav rules: flexbox with `justify-content: space-between`, wrapping for mobile. Style `.nav-title` (bold, no underline) and `.nav-links` (flex row, gap, subdued color, hover state).

### `assets/css/components.css`

Add within `@layer components`:
- `.pagination` — flex centered, disabled state
- `.category-title` — category page heading
- Pagefind UI CSS variable overrides mapping `--pagefind-ui-*` to existing design tokens. These adapt to dark/light automatically since they reference our responsive token variables.

---

## Phase 6 — GitHub Actions

### `.github/workflows/pages.yml`

Add Node.js setup and Pagefind step:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: "24"

# after Jekyll build:
- name: Index with Pagefind
  run: npx pagefind@latest --site _site
```

---

## Phase 7 — Dev-Blog Skill Updates

### `SKILL.md`

- Add `"Bash(npx pagefind *)"` to allowed-tools
- Update content type table: rename `update`/`concept` → `progress`/`engineering`, add note that categories are open-ended
- Add `announcement` and `future` as anticipated categories in the table

Updated table:

| Type | Category | Purpose |
|------|----------|---------|
| Progress | `progress` | Development progress, new capabilities, strategic direction |
| Engineering | `engineering` | Technical architecture and design documents |
| Announcement | `announcement` | Brief, focused announcements of specific details |
| Future | `future` | Speculative engineering concepts explored for the joy of it |

Note in SKILL.md that this table is non-exhaustive — the user can introduce new categories at draft time.

### `commands/draft.md`

- Step 2 "Determine Content Type": replace fixed 2-option table with dynamic approach. List categories from existing posts as options, allow user to specify a new one. If the style profile has per-category guidance, use it; otherwise use the general voice calibration.
- Update structural pattern references from "Update"/"Concept" to "Progress"/"Engineering"
- Add structural patterns for `announcement` and `future`

### `commands/publish.md`

- Step 6: add `npx pagefind --site _site` to build verification

### `commands/init.md`

- Step 3: add new template files to copy list (`category.html`, `pagination.html`, `search.html`)
- Step 5: add `bundle add jekyll-paginate-v2`
- Update template file references

### Style profile (`.claude/context/style-profile.md`)

Rename section headers:
- "Update Posts" → "Progress Posts" throughout
- "Concept Documents" → "Engineering Posts" throughout

Add stub sections for anticipated categories:
- **Announcement Posts**: Brief, direct, factual. 1-2 paragraphs max. States the specific detail and its immediate implication. No deep technical dive.
- **Future Posts**: Exploratory, speculative tone. First-person musing. Longer-form. Permission to be imaginative while maintaining technical grounding. The audience is the author and fellow engineers who enjoy thinking about what's possible.

### Template files in `.claude/skills/dev-blog/template/`

Update all corresponding template files to match live blog changes:
- `template/_layouts/category.html` (new)
- `template/_includes/pagination.html` (new)
- `template/_includes/nav.html` (updated — dynamic categories)
- `template/_layouts/post.html` (updated — Pagefind attrs)
- `template/assets/css/layout.css` (updated)
- `template/assets/css/components.css` (updated)
- `template/github-workflows/pages.yml` (updated — Node.js + Pagefind)
- `template/search.html` (new)
- `template/_config.yml` (updated — pagination/autopages config)
- `template/CLAUDE.md` (updated)

### `.claude/CLAUDE.md`

- Update content type descriptions (progress/engineering)
- Add entries for pagination, category pages, search
- Note dynamic category system

---

## Files Summary

| Action | File | Phase |
|--------|------|-------|
| Modify | `_posts/2026-02-07-consolidating-the-kernel-monorepo.md` | 1 |
| Modify | `_posts/2026-02-07-introducing-the-dev-blog.md` | 1 |
| Modify | `_posts/2026-02-07-dev-blog-architecture.md` | 1 |
| Modify | `_posts/2026-02-07-kernel-architecture-overview.md` | 1 |
| Modify | `Gemfile` | 2 |
| Modify | `_config.yml` | 2 |
| Create | `_layouts/category.html` | 3 |
| Create | `_includes/pagination.html` | 3 |
| Modify | `_includes/nav.html` | 3 |
| Modify | `_layouts/post.html` | 3 |
| Create | `search.html` | 4 |
| Modify | `assets/css/layout.css` | 5 |
| Modify | `assets/css/components.css` | 5 |
| Modify | `.github/workflows/pages.yml` | 6 |
| Modify | `.claude/CLAUDE.md` | 7 |
| Modify | `.claude/context/style-profile.md` | 7 |
| Modify | `.claude/skills/dev-blog/SKILL.md` | 7 |
| Modify | `.claude/skills/dev-blog/commands/draft.md` | 7 |
| Modify | `.claude/skills/dev-blog/commands/init.md` | 7 |
| Modify | `.claude/skills/dev-blog/commands/publish.md` | 7 |
| Create | `.claude/skills/dev-blog/template/_layouts/category.html` | 7 |
| Create | `.claude/skills/dev-blog/template/_includes/pagination.html` | 7 |
| Create | `.claude/skills/dev-blog/template/search.html` | 7 |
| Modify | `.claude/skills/dev-blog/template/_includes/nav.html` | 7 |
| Modify | `.claude/skills/dev-blog/template/_layouts/post.html` | 7 |
| Modify | `.claude/skills/dev-blog/template/_config.yml` | 7 |
| Modify | `.claude/skills/dev-blog/template/CLAUDE.md` | 7 |
| Modify | `.claude/skills/dev-blog/template/assets/css/layout.css` | 7 |
| Modify | `.claude/skills/dev-blog/template/assets/css/components.css` | 7 |
| Modify | `.claude/skills/dev-blog/template/github-workflows/pages.yml` | 7 |

---

## Verification

1. `bundle install` — jekyll-paginate-v2 installs
2. `bundle exec jekyll build` — build succeeds; `_site/categories/progress/index.html` and `_site/categories/engineering/index.html` exist
3. `npx pagefind --site _site` — indexing completes, `_site/pagefind/` created
4. `bundle exec jekyll serve` — visual checks:
   - Home page lists all 4 posts with new category labels
   - Nav shows "Engineering", "Progress", "Search" links (dynamic from `site.categories`)
   - `/categories/progress/` shows 2 progress posts
   - `/categories/engineering/` shows 2 engineering posts
   - Post URLs use new category names (`/progress/...`, `/engineering/...`)
5. Search (serve `_site` statically after Pagefind indexing):
   - `/search/` loads with input field
   - Query returns results with `/jaime/`-prefixed links
   - Category and tag facet filters appear with new category names
6. Playwright screenshots (via `playwright-cli screenshot --filename=_capture/.playwright/<name>.png`) of home, both category pages, a post, and search
7. Dark/light mode verification for all new components
