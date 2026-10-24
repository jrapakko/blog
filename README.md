# Blog

Static blog built with [Zola](https://www.getzola.org/) and the [zola-quiet](https://github.com/johnnybravo-xyz/zola-quiet) theme (git submodule in `themes/`), deployed to GitHub Pages via Actions.

After cloning: `git submodule update --init`.

## Write a post
Add `content/posts/my-post.md`:

    +++
    title = "My post"
    date = 2026-10-24
    +++

    Body in Markdown.

Push to `main` and it deploys.

## Preview locally
Zola isn't in Fedora's repos; grab the binary from https://github.com/getzola/zola/releases into `~/.local/bin`.

    zola serve    # http://127.0.0.1:1111

## First deploy
1. Push to a GitHub repo.
2. Settings → Pages → Source: **GitHub Actions**.

## Custom domain (later)
1. Settings → Pages → Custom domain → enter your domain. With Actions deploys, no `CNAME` file is needed;
   the build picks up the new URL automatically.
2. DNS:
   - Subdomain: `CNAME` record → `<username>.github.io`
   - Apex: `A` records → 185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153
     (optional `AAAA`: 2606:50c0:8000::153, 2606:50c0:8001::153, 2606:50c0:8002::153, 2606:50c0:8003::153)
3. Once DNS resolves, tick **Enforce HTTPS**.
