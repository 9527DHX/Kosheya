---
title: 使用 Github Actions 和 Netlify/Vercel 实现自动部署 Hexo 博客
date: 2024-08-22 14:53:18
thumbnail: "https://api.miaomc.cn/image/get"
cover: "https://api.miaomc.cn/image/get"
---
以下代码修改自 Hexo 官方文档：[在 GitHub Pages 上部署 Hexo](https://hexo.io/zh-cn/docs/github-pages)，本教程需要结合上述或其他教程使用。
使用前，请先在仓库中建立 ``.github/workflows/xxx.yml``，名字你喜欢。

这个工作流文件主要实现了以下流程：
用户在本地修改 Hexo 内的文章，将更改同步到 Github，触发 Github Actions，根据 ``.github/workflows/xxx.yml`` 生成新的静态文件，并将文件更新到同仓库内的 `gh-pages` 分支（可自定义），而  Netlify/Vercel 则可通过预先的设置拉取 `gh-pages` 分支并直接进行网站的部署。

相比其他方式，本方法将构建的过程搬到了 Github Actions 上进行。
最后一个步骤使用的 `peaceiris/actions-gh-pages` 这个工作流预设还有更多的玩法，详情可查阅：[GitHub Pages action - GitHub Marketplace](https://github.com/marketplace/actions/github-pages-action)

```
name: Build and Deploy # 可自定义工作流的名称

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          submodules: recursive
          
      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache

      - name: Install Dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public # 可自定义Hexo输出的目录

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public # 可自定义Hexo输出的目录
          publish_branch: gh-pages # 可自定义输出的分支
```