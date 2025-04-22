+++
title = "My blogging workflow: based on Emacs"
author = ["SmartAI"]
date = 2025-04-21T19:39:00-07:00
tags = ["blog", "emacs", "hugo"]
draft = false
+++

During the holiday at home these past few days, I decided to pick up `Emacs` again. So, I went ahead and completely reconfigured my `Emacs` setup, covering both my usual writing ( `Org` , `LaTeX` , etc.) and programming (mainly `C++` ). After finishing this setup tinkering, I remembered I had an unused domain name I wanted to utilize, so I decided to build this blog. Actually, I had been considering setting up a blog for a while but always gave up, mainly due to a lack of motivation for writing. Over the weekend, on the way to `Parksvill` , I read Li Xiaolai's book ["The Truth of Attention"]({{< relref "the-truth-of-attention" >}}), which made me feel it's necessary to strengthen my habit of written output.

<!--more-->

This blog post briefly documents how I set up this workflow, mainly involving several parts: configuration in `Emacs` , usage of `hugo` and its templates, and using `GitHub Pages` .

I am using the combination of Emacs + Hugo + Github Pages. Since I'm quite used to writing Org documents, after writing my blog posts in Org within Emacs, I directly use `ox-hugo` to export them to Markdown, and then push them to GitHub for serving. Here, I primarily referred to the `ox-hugo` [official documentation](https://ox-hugo.scripter.co/).
The configuration in `Emacs` is mainly to handle the export from Org files to Markdown files. I adopted the 'One post per Org subtree' approach, where each article is a second-level heading. Each first-level heading represents a separate section. There isn't much to say about this part; just configure and organize your Org documents according to the official documentation. One thing to pay special attention to is that the top of the Org file needs the `hugo_base_dir` configured, and each post needs its export filename configured via `EXPORT_FILE_NAME` . Once configured, you can write your blog posts just like you normally write Org documents. Theoretically, all formats supported by Org can be used directly, but be aware that some formats might produce unexpected behavior when exporting to Markdown. When you need to export, you can use the Emacs shortcut `C-c C-e H H` to export to a Markdown document.

To use Hugo to generate the static site, you can install the environment locally and preview the generated web pages. For Hugo installation, refer to the official documentation. Here I recommend a very simple theme, [hugo-bear](https://themes.gohugo.io/themes/hugo-bearblog/); configure it according to its documentation. After configuration, run `hugo server` locally to ensure you can successfully preview the website.

To serve the website online, there are many options. There are paid and free solutions, such as `cloudflare` or `GitHub Pages` . I personally use `GitHub Pages` . `GitHub Pages` is essentially a service for serving static files. Therefore, before that, you still need to use Hugo to generate the static files from the Markdown files. Since `GitHub` has the `Action` feature, it's very convenient to automatically generate these static files after we `Push` the code to the `remote` . As for configuring the `Action` , I directly asked `ChatGPT` , and the provided script can be used directly. Save this script to the `.github/workflows/deploy.yml` file in your blog's code repository.

```yaml
  name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch (e.g., main)
  push:
    branches:
      - main

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
      HUGO_VERSION: 0.146.6
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3 # Upload the built html files
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build #
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4 #
```

Additionally, you need to go into the repository's `setting` to enable Pages using `Actions`. You can also configure your custom domain there. Refer to this screenshot:
![](/images/github-page.png)


## Summary {#summary}

At this point, the entire workflow should be up and running. This very article was published using this process. To summarize, here are the key steps for writing a blog using `Emacs`:

Configure `ox-hugo` and organize your `org` files according to the documentation requirements.
Install and configure the `hugo` site locally, including the `theme`.
Configure `Github Pages` and set up a custom domain.
Configure the `Github` workflow to support automatic publishing after a `Git Push`.
After that, you can write entirely within `Emacs` and conveniently publish your text as online articles. Going forward, there is still room for optimization, including using `Org mode`'s `Capture template` feature to quickly create posts, configuring different versions for Chinese and English, and adjusting styles, etc. I hope this article is helpful for friends with similar needs. You can also directly refer to my complete source code: [Blog Source Code](https://github.com/SmartAI/myblog)


## References {#references}

-   <https://ox-hugo.scripter.co/>
-   <https://github.com/SmartAI/myblog>
