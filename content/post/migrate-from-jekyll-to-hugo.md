---
title: "Migrate from Jekyll to Hugo"
date: 2018-02-22T17:29:13+08:00
draft: true
---

Today I spent around one hour to switch my blog from `Jekyll` to `Hugo`. The reason is pretty simple.

1. I've always wanted to enter the `golang` world, so Hugo would be a great place to start
2. Disqus and staticman just did not work with Jekyll
3. Jekyll has been there so long enough and now it's time try something new (and faster)

The swtich is surprisingly easy and straight-forward.

## Migration

Hugo provides a very handy official command line tool [hugo_import_jekyll][hugo_import_jekyll].

```bash
hugo import jekyll jekyll_root_path hugo_root_path
```

This will help import all the posts, static assets, migrate the front matter and some markdown syntax. (like syntax highlighting)

[hugo_import_jekyll]: https://gohugo.io/commands/hugo_import_jekyll/

## The Theme

In the Hugo world, a theme is simply a git submodule linked to the theme repo. (unlike jekyll, which is a ruby gem)

There are just so many [themes][themes] to choose from. I am instantly hooked by [hugo-nuo][hugo-nuo] because of its simplicity and great Chinese support. (网易云播放器等等)

### Install the theme

```bash
git submodule add https://github.com/laozhu/hugo-nuo themes/hugo-nuo
```

and in `config.toml`

```toml
theme = "hugo-nuo"
```

Within minutes, I have copied the About and Project pages from my old Jekyll site.

[themes]: https://themes.gohugo.io/
[hugo-nuo]: https://github.com/laozhu/hugo-nuo

## Clean-up

### GitHub flavored syntax highlighting

Because I am so used this way of doing code highlighting and the jekyll-to-hugo conversion tool did not work for all my old posts. Enabling `pygmentsCodeFences=true` will do the trick and all GitHub flavored syntax worked just well.

```toml
# GitHub flavored syntax highlighting
pygmentsCodeFences=true
```

## Deployment

The new site still deployed to the awesome `Netlify`. (Again, it is free for personal, and so much better than GitHub Pages) I just need to add a `netlify.toml` and everything else is the same as in the [jekyll tutorial][jekyll-tutorial].

[jekyll-tutorial]: {{< ref "2017-10-09-creating-your-personal static blog with jekyll.md" >}}

```toml
[build]
publish = "public"
command = "hugo"

[context.production.environment]
HUGO_VERSION = "0.36.1"
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"

[context.deploy-preview]
command = "hugo -b $DEPLOY_PRIME_URL"

[context.deploy-preview.environment]
HUGO_VERSION = "0.36.1"

[context.branch-deploy]
command = "hugo -b $DEPLOY_PRIME_URL"

[context.branch-deploy.environment]
HUGO_VERSION = "0.36.1"

[context.next.environment]
HUGO_ENABLEGITINFO = "true"
```

## Profit

Phew! That's it. My new site is now built with `Hugo`. One thing I really love about Hugo is it has LiveRelaod built-in. Isn't that just great?
