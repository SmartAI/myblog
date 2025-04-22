+++
title = "我的博客写作工具流-基于Emacs"
author = ["SmartAI"]
date = 2025-04-21T19:39:00-07:00
tags = ["blog", "emacs", "hugo"]
draft = false
+++

这几天放假在家决定重拾一下 `Emacs` ，索性重新配置了一遍完整的 `Emacs`  配置，包括平时的写作（ `Org` ， `LaTeX` 等）以及编程（主要是 `C++` ）。折腾完毕之后又想起自己有个闲置的域名想用起来，所以就准备搭建这个博客。其实以前一直在考虑搭建个博客，但是总是放弃了，主要原因还是没有写作的动力。周末在去 `Parksvill` 的途中看了李笑来的书[《专注力的真相》]({{< relref "the-truth-of-attention" >}})，让我觉得还是有必要加强自己的文字输出的习惯的。

<!--more-->

这篇博文就简单记录一下我是如何搭建这个工作流的，主要是设计到几个部分： `Emacs` 中的配置， `hugo` 的使用以及模板， `GitHut Page` 的使用。

我使用的是 Emacs + Hugo + Github Pages 这个组合。因为我比较习惯写 org 文档，所以我在 Emacs 写完了 org 的博客之后直接使用 `ox-hugo` 导出到 markdown 然后 push 到 Github serving 即可。这里主要参考 `ox-hugo` 的[官方文档](https://ox-hugo.scripter.co/)。
`Emacs` 中配置主要是为了解决从 org 文件导出到 md 文件问题，我才用的是 One post per Org subtree 的方式，每一篇文章都是一个二级标题。每个一级标题都是一个单独的 section。这部分没啥好说的，根据官方文档来配置和组织自己的 org 文档就好了。需要特别注意的是 org 文件顶部需要配置 `hugo_base_dir` ，每篇文章需要配置导出的文件名称 `EXPORT_FILE_NAME` 。配置好了之后就可以用平时写 org 文档的方式写自己的博客文章了。理论上 org 支持的各种格式都是可以直接用，但是需要注意导出 markdown 的时候部分格式可能会产生非预期的行为。需要导出的时候可以使用 Emacs 中的快捷键 `C-c C-e H H` 导出到 markdown 文档。

为了使用 Hugo 生成静态网站，可以在本地安装环境和预览生成的网页。关于 Hugo 的安装可查看官方文档。这里我推荐一个非常简单的模板 [hugo-bear](https://themes.gohugo.io/themes/hugo-bearblog/) , 按照文档配置即可。配置完成后在本地运行 `hugo server` 确保本地可以成功预览网页。

为了网站能够在线 serving，有非常多的选择。有一些付费的也有免费的方案，比如 `cloudflare` 或者 `GitHub Pages` ，我自己使用的是 `GitHub Pages` 的方式。 `GitHub Pages` 本质上是 serving 静态文件的服务。所以在这之前还需要使用 hugo 来根据 md 文件生成静态文件。由于 `GitHub` 有 `Action` 的功能，这样就很方便在我们 `Push` 代码到 `remote` 之后自动生成这个静态文件。至于 `Action` 的配置方法，我直接问了一下 `ChatGPT` ，这个脚本直接可以使用。保存这个脚本到博客代码仓库的 `.github/workflows/deploy.yml` 文件中。

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

另外在需要在代码仓库的 `setting` 中开启 pages 使用 =Actions=，同时可以在里面配置自己的域名，可以参考这个截图：
![](/images/github-page.png)


## 总结 {#总结}

至此，整个流程应该是跑通了，这篇文章就是通过这个流程发布的。总结一下使用 Emacs 写 blog 几个关键步骤：

-   配置 ox-hugo，并且按照文档要求组织 org 文件
-   本地安装和配置 hugo 网站，包括 theme
-   配置 Github Pages，自定义域名
-   配置 Github 工作流，支持 Git Push 之后自动发布

    再之后就完全可以在 Emacs 中写文字然后方便发布成在线文章了。接下来还有一些优化空间，包括使用 Org mode 的 Capture tempalte 功能来快创建文章，配置中英文不同的版本，以及一些样式等等。希望这篇文章对有同样需求的朋友有帮助，你也可以直接参考我的全部源代码：[博客源代码](https://github.com/SmartAI/myblog)


## References {#references}

-   <https://ox-hugo.scripter.co/>
-   <https://github.com/SmartAI/myblog>
