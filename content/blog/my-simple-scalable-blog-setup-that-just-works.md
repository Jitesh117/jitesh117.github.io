+++
title = "My Simple Blog Setup That Just Works, and Scales!"
date = 2025-03-27T11:44:25+05:30
draft = false
showToc = true
cover.image = 'images/blog_setup.png'
tags = ["blog"]
+++

Ever since I started posting the analytics of my blog, people keep asking me about my blog setup. So I decided to write a blog on it, which you're reading currently haha.

When I decided that I want to start a blog, I was in a dilemma, should I go with platforms like Medium, Hashnode, etc or build my own site. After consulting with many people, I was told a lot of things, like:

> Personally, I just associate Medium, Hashnode & DevTo with crappy content now. There might be a few good authors there but I'm more often disappointed than not.

> It takes very little effort to deploy a site with some template on Github Pages or any other static hosting site.

> Medium is low signal

After hearing these opinions, I decided I should go with building my own site. But I didn't want to spend a lot of time in just customising the look of my website. I wanted something quick to spin-up and easy to modify on my will.

I spent a lot of time researching Static Site Generators (SSGs) and stumbled upon Hugo. It looked quite promising so I went with it and decided to go with the Papermod theme, since it looks quite minimalist with main focus on the content.

If you also want a no-fluff easy to setup blog that scales, you're at the right place. Let me walk you through my blogging setup.

{{< callout type = "info">}}
If you want to see my code for my blog, here's the [github repo link](https://github.com/Jitesh117/jitesh117.github.io/).
{{< /callout >}}

## Getting Started With Hugo

First of all, you should install the Hugo framework. To install Hugo, check out the [installation](https://gohugo.io/installation/) page of the docs, depending on your Operating System.

### Create a site

{{< callout type = "warning">}}
If you're a windows user, **DO NOT** use command prompt or Windows Powershell, use a linux terminal such as WSL or Git Bash.
{{< /callout >}}

```bash
hugo new site MyFreshWebsite --format yaml
# replace MyFreshWebsite with name of your website
```

### Installing/Updating a Theme

There's a plethora of options in the Hugo Theme Marketplace. I use the Papermod theme.

- Themes reside in MyFreshWebsite/themes directory.
- PaperMod will be installed in MyFreshWebsite/themes/PaperMod

There are multiple methods to install a theme, but the recommended way is to install it via a git submodule, which comes in handy when you want to update the theme with the latest release.

**NOTE**: Inside the folder of your Hugo site `MyFreshWebsite`, run this:

```bash
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```

Finally, set-up the theme as PaperMod in your site config:
In `hugo.yaml`, add this:

```yaml
theme: ["PaperMod"]
```

## Configuring PaperMod Theme

Now that you've got Hugo installed and the PaperMod theme added as a submodule, let's configure it to make it your own.

### Basic Configuration

Edit your `hugo.yaml` file to include these basic configurations:

```yaml
baseURL: "https://yourusername.github.io/"
title: "Your Blog Title"
paginate: 5
theme: ["PaperMod"]

enableRobotsTXT: true
buildDrafts: false
buildFuture: false
buildExpired: false

minifyOutput: true

params:
  env: production
  title: Your Blog Title
  description: "Your blog description here"
  keywords: [Blog, Portfolio, PaperMod]
  author: Your Name
  # author: ["Me", "You"] # multiple authors
  images: ["<link or path of image for opengraph, twitter-cards>"]
  DateFormat: "January 2, 2006"
  defaultTheme: auto # dark, light
  disableThemeToggle: false

  ShowReadingTime: true
  ShowShareButtons: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  ShowWordCount: true
  ShowRssButtonInSectionTermList: true
  UseHugoToc: true
  disableSpecial1stPost: false
  disableScrollToTop: false
  comments: false
  hidemeta: false
  hideSummary: false
  showtoc: false
  tocopen: false

  assets:
    # disableHLJS: true # to disable highlight.js
    # disableFingerprinting: true
    favicon: "<link / abs url>"
    favicon16x16: "<link / abs url>"
    favicon32x32: "<link / abs url>"
    apple_touch_icon: "<link / abs url>"
    safari_pinned_tab: "<link / abs url>"

  label:
    text: "Home"
    icon: /apple-touch-icon.png
    iconHeight: 35

  # profile-mode
  profileMode:
    enabled: false # needs to be explicitly set
    title: Your Name
    subtitle: "Your bio here"
    imageUrl: "<img location>"
    imageWidth: 120
    imageHeight: 120
    imageTitle: my image
    buttons:
      - name: Posts
        url: posts
      - name: Tags
        url: tags

  # home-info mode
  homeInfoParams:
    Title: "Hi there \U0001F44B"
    Content: Welcome to my blog

  socialIcons:
    - name: twitter
      url: "https://twitter.com/"
    - name: github
      url: "https://github.com/"
    - name: linkedin
      url: "https://linkedin.com/in/"

  cover:
    hidden: true # hide everywhere but not in structured data
    hiddenInList: true # hide on list pages and home
    hiddenInSingle: true # hide on single page

  # for search
  # https://fusejs.io/api/options.html
  fuseOpts:
    isCaseSensitive: false
    shouldSort: true
    location: 0
    distance: 1000
    threshold: 0.4
    minMatchCharLength: 0
    keys: ["title", "permalink", "summary", "content"]

menu:
  main:
    - identifier: categories
      name: Categories
      url: /categories/
      weight: 10
    - identifier: tags
      name: Tags
      url: /tags/
      weight: 20
    - identifier: archives
      name: Archives
      url: /archives/
      weight: 30
    - identifier: search
      name: Search
      url: /search/
      weight: 40
```

### Creating Content

Now let's create your first blog post:

```bash
hugo new content posts/my-first-post.md
```

This will create a new file at `content/posts/my-first-post.md` with some default front matter like this:

```markdown
---
title: "My First Post"
date: 2025-03-23T12:00:00+05:30
draft: true
---

Write your content here...
```

Update the front matter as needed and set `draft: false` when you're ready to publish.

### Customizing Your Posts

PaperMod supports various front matter parameters for posts. Here's an example:

```markdown
---
title: "My Amazing Post"
date: 2025-03-23T12:00:00+05:30
draft: false
description: "Description text here"
tags: ["html", "css", "markdown"]
categories: ["web"]
showToc: true
weight: 1
cover:
  image: "cover.jpg"
  alt: "This is a post image"
  caption: "This is the caption"
  relative: false
---
```

### Testing Locally

Before deploying, test your site locally:

```bash
hugo server -D
```

This will start a local server (usually at http://localhost:1313/) with draft posts included (`-D` flag).

## Deploying to GitHub Pages

Now let's set up GitHub Pages deployment using GitHub Actions. This workflow automates the deployment process, ensuring that whenever you push changes to the master branch, your site is rebuilt and published via GitHub Pages.

### Creating a GitHub Repository

1. Create a new repository on GitHub (e.g., `yourusername.github.io` or any name you prefer)
2. Push your local repository to GitHub:

```bash
git remote add origin https://github.com/yourusername/your-repo-name.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

### Setting Up GitHub Actions Workflow

Create a file in your repository at `.github/workflows/hugo.yml` and paste the GitHub Actions workflow you provided:

```yaml
name: Deploy Hugo site to Pages
on:
  push:
    branches:
      - master
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: "pages"
  cancel-in-progress: false
defaults:
  run:
    shell: bash
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.123.8
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "lts/*"
      - name: Install Dart Sass
        run: npm install -g sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v4
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
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

### Enable GitHub Pages

1. Go to your repository settings on GitHub
2. Navigate to "Pages" in the sidebar
3. Under "Build and deployment", set the source to "GitHub Actions"
4. Commit and push your workflow file, and GitHub Actions will automatically build and deploy your site

## Advanced PaperMod Features

### Search Functionality

To add search functionality to your blog:

1. Create a search page:

```bash
hugo new content search.md
```

2. Add this front matter:

```markdown
---
title: "Search"
layout: "search"
---
```

3. Make sure you have the search menu item in your config file (already added in the example above).

### Custom CSS

If you want to customize the CSS:

1. Create a file at `assets/css/extended/custom.css`
2. Add your custom CSS there, which will be loaded after the theme's CSS

```css
/* Custom CSS */
:root {
  --primary-color: #1e90ff;
}

.post-title {
  font-size: 2.5rem;
}
```

### Analytics Setup

To add Google Analytics, add this to your `hugo.yaml`:

```yaml
googleAnalytics: G-MEASUREMENT_ID
```

## Updating Your Blog

### Adding New Posts

Simply create new files in the `content/posts/` directory:

```bash
hugo new content posts/another-great-post.md
```

### Updating the Theme

To update the PaperMod theme to the latest version:

```bash
git submodule update --remote --merge
```

## Conclusion

That's it! You now have a minimalist, fast, and scalable blog setup with Hugo and the PaperMod theme, deployed automatically to GitHub Pages whenever you push changes. This setup gives you:

- Full control over your content
- Great performance (static sites are super fast)
- No hosting costs (thanks to GitHub Pages)
- Easy content management with Markdown
- A professional look with minimal effort

This is the exact setup I use for my blog, and it has served me well. It's simple enough that I can focus on writing content rather than maintaining the website itself, yet flexible enough to grow with my needs.

Feel free to reach out if you have any questions or need help setting up your blog!
