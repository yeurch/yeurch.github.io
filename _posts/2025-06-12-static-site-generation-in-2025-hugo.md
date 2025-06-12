---
layout: post
title:  "Static Site Generation in 2025 - Hugo"
---
# Static Site Generation in 2025

As I'm writing, this site is hosted on GitHub Pages, using Jekyll, which is supported out of the box.  I had a chat with my good friend ChatGPT, who suggests that Jekyll is not the only game in town.  It threw up Hugo and Gatsby as decent alternatives, with the following bulleted notes.

### Hugo
- Language: Go
- Speed: Extremely fast — one of the fastest SSGs.
- Templates: Go templating (a bit verbose but very flexible).
- Content Format: Markdown, TOML/YAML front matter.
- Strengths:
  - Very fast build times (even for large sites).
  - Good multilingual support.
  - Easy to install (single binary).
- Weaknesses:
  - Go templating syntax is not intuitive for beginners.
  - Less vibrant plugin ecosystem than Gatsby or Jekyll.
- Best For: Large blogs, documentation sites, multilingual content.

### Gatsby
- Language: JavaScript/React
- Speed: Slower builds for large sites, though incremental builds improve this.
- Templates: React components.
- Content Format: Markdown, CMS, APIs, databases.
- Strengths:
  - Rich plugin ecosystem and GraphQL data layer.
  - Integrates well with headless CMSs (Contentful, Sanity, etc.).
  - Progressive web app features out of the box.
- Weaknesses:
  - Steeper learning curve for non-React developers.
  - Build performance issues with large sites.
- Best For: Modern web apps, marketing sites, headless CMS integration.

### Jekyll
- Language: Ruby
- Speed: Slower than Hugo but fine for small to medium sites.
- Templates: Liquid templating.
- Content Format: Markdown, YAML front matter.
- Strengths:
  - Long-established, with a large user base.
  - Officially supported by GitHub Pages.
  - Simple and well-documented.
- Weaknesses:
  - Ruby dependency can be troublesome on non-Ruby environments.
  - Slow for large sites or complex plugins.
- Best For: Blogs, personal websites, GitHub Pages users.

While Jekyll and GitHub Pages have great integration, it should be quite straight-forward to deploy sites created with other static site generation tools to GitHub Pages using GitHub Actions.  I thought I'd take them for a spin, starting with Hugo.  In future posts, I'll explore Gatsby, and also revisit Jekyll to see what the experience is like these days.

## Hugo

Hugo, available at https://gohugo.io, bills itself as "the world's fastest framework for building websites".

Installation was honestly a little confusing.  First up, Hugo is available in three editions: standard, extended, and extended/deploy.  The [quick start](https://gohugo.io/getting-started/quick-start/) says I need the extended or extended/deploy edition.  It seems a little odd that the quick start needs more than the standard edition.  Apparently, Homebrew (I'm running MacOS) installs the extended version of Hugo.  This makes me feel that the extended edition is really the standard edition.

```bash
$ brew install hugo
$ hugo version
hugo v0.147.8+extended+withdeploy darwin/arm64 BuildDate=2025-06-07T12:59:52Z VendorInfo=brew
```

Oh, ok, so it seems I got the extended/deploy version.  Whatever.  This is a bit of a mess. 🤔

Next up is the choice of Sass transpiler.  Extended is needed to transpile Sass to CSS using its embedded LibSass transpiler.  The Dart Sass transpiler can be used with any edition.  On the face of it, that sounds pretty straight-forward, i.e. Sass transpilation is included with the extended edition.  But then, in [Hugo's Sass documentation](https://gohugo.io/functions/css/sass/#dart-sass), it says:

> Hugo's extended and extended/deploy editions include LibSass to transpile Sass to CSS. In 2020, the Sass team deprecated LibSass in favor of Dart Sass.

Hmm. So I can install the extended edition to get deprecated Sass support out of the box.  Weird.  I installed Dart Sass on my system just to make sure I wasn't going to be using anything deprecated (although I later found that the quick start doesn't seem to require Sass, so this wasn't a real issue).

> brew install sass/sass/sass

At this point, I was pretty down on Hugo, I'll be honest.  Things changed.  Once I got to `hugo new site quickstart` and beyond, things were great!

The availability of themes seems really strong, and they're easily managed by installing with git's submodule support, and the active theme is specified in the site's TOML configuration file (while we're here, +1 for using TOML as default, rather than just YAML).  Theming my site was as simple as:

```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo "theme = 'ananke'" >> hugo.toml
```

Adding content was very straightforward too, with `hugo new content <path_to_content>`. The generated content file had straight-forward and easy to understand front matter.  New pages are marked as draft by default, and it's trivial to included draft content when developing locally by running Hugo with `hugo server -D`.

Which brings us onto Hugo's dev server.  It does everything you'd expect it to do ... automatically picking up new content changes as soon as they're saved to disk, and even causing the web page previewing your site to automatically refresh too.  And this reload is _blazingly_ fast.

### Deploying Using GitHub Actions

While not as simple as GitHub's build in support for Jekyll, it's fairly straight-forward to use GitHub Actions to support deployment of a Hugo site.

I already have my main GitHub Pages repository, `yeurch.github.io` setup as https://richardfawcett.net, so any other repo I setup to use GitHub Pages will automatically get deployed as a subdirectory of that site (unless I over-ride it with a custom domain).  So I setup a repository [hugo-quickstart](https://github.com/yeurch/hugo-quickstart) which, once configured for Pages, will deploy [here](https://richardfawcett.net/hugo-quickstart).

I pushed my changes to the repo, and went to the repository settigns and chose "Pages" in the left-sidebar.  I changed the rule from deploy from a branch to deploy using GitHub Actions, and I was presented with a list of possible actions.  Hugo wasn't immediatley presented as a suggestion, but I searched through the available actions, and there _was_ a suitable action for deployment.  GitHub guided me through committing this to my repo as file `.github/workflows/hugo.yml`, and it _just works_.  Here's the content of the yml file:

```yml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.128.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

Committing this file automatically triggered itself, and it deployed my site as expected.  One issue ... my post was still marked as draft, so I authored [another commit](https://github.com/yeurch/hugo-quickstart/commit/957f8a2fe0aaf8c91c6a75585f9b70fe3a07f942) to properly publish it, pushed this up to GitHub, and it was reflected in my pages site within seconds.
