---
author_profile: true
categories:
- Tech
comments: true
date: 2017-10-09T00:00:00Z
last_modified_at: 2018-02-21 17:30:25 +08:00
read_time: true
related: true
share: true
tags:
- Mac
- Ruby
- Jekyll
- Blog
- Tutorial
title: Creating your personal static blog with jekyll
---

By the end of this tutorial, you will have your own personal blog :tada:

* Created by [static site generator][staticgen]
* Hosted for free on [netlify][netlify]
* Custom domain like _yourdomain.com_
* Secured by HTTPS by [Let's Encrypt][letsencrypt]

## Tools used in this tutorial

* [jekyll][jekyll]
* [rbenv][rbenv] (for managing ruby environment)
* a text editor (Sublime Text or vscode)
* (optional) a domain of your own, purchased separately from sites like GoDaddy

## Environment setup

Since the static site generator used here **jekyll** is written in Ruby, we will need install a Ruby development.

### System default Ruby

If you are on a Mac, ruby should be installed by the system already. You can quickly verify by:

```bash
$ which ruby
/usr/bin/ruby

$ ruby --version
ruby 2.3.3p222 (2016-11-21 revision 56859) [universal.x86_64-darwin17]
```

However, I like to install a user-maintained version of Ruby, so that I can install multiple gems without touching the system environment. **rbenv** is the tool for that, which allows you to install multiple versions of ruby and easily manage them. You can also use [rvm][rvm]. It is really a personal choice.

### Install rbenv

If you are on a Mac, you can use brew and it can do all the magic for you.

```bash
# Install rbenv
brew install rbenv
```

### Install latest version of Ruby

```bash
# List all available versions to install, you will see a huge list but I will just grab
# the latest version at the time of writing is 2.4.2
$ rbenv install --list

# Install ruby 2.4.2
$ rbenv install 2.4.2

# Run this after you install a new version of Ruby
$ rbenv rehash

# See all installed versions
$ rbenv versions
* system
2.4.2 (set by /Users/liuyang/.rbenv/version)

# Set 2.4.2 to be the default
$ rbenv global 2.4.2
system
* 2.4.2 (set by /Users/liuyang/.rbenv/version)

# Verify default is set to 2.4.2
$ which ruby
/Users/liuyang/.rbenv/shims/ruby

$ ruby --version
ruby 2.4.2p198 (2017-09-14 revision 59899) [x86_64-darwin17]
```

### Install jekyll

Now that we have ruby successfully installed by rbenv, let's install the **jekyll** gem (a Ruby plugin/library).

```bash
# Install two gems jekyll and bundler
gem install jekyll bundler

# Create a new jekyll app
$ jekyll new my-blog
$ cd my-blog
$ bundle exec jekyll serve
```

Here you go, you have your first jekyll site up and running at `http://localhost:4000`.

Note that changes made are automatically reloaded, so you don't have to re-start the server. (not applicable to*\_config.yml*)

## Install the minimal mistakes theme

The default jekyll theme [minima][mimina] is great, but I just love the [minimal mistake][mmistakes] theme designed by Michael Rose. It is both minimalistic and fully customizable.

Add the theme in you `Gemfile`:

```ruby
gem "minimal-mistakes-jekyll"
```

Set the theme in `_config.yml`:

```yml
theme: minimal-mistakes-jekyll
```

Run Bundler to install the theme gem and dependencies:

```bash
bundle install
```

Finally, a few clean-up:

1. Replace `<site root>/index.md` with a modified Minimal Mistakes [index.html][index.html]
2. Change layout: `post` in `_posts/0000-00-00-welcome-to-jekyll.markdown` to `layout: single`
3. Remove `about.md`

Restart jekyll server to view the changes.

## Set up Github repository

Now let's create a repository in your Github called `USERNAME.github.com`, like for me it's `sfdye.github.com`. If you have an existing repo, I suggest your make a backup and delete the repo to start from scratch.

To commit our jekyll app to the GitHub repository we just created:

```bash
git remote add origin git@github.com/USERNAME/USERNAME.github.com.git
git add .
git commit -m "First commit, Jekyll is awesome"
git push -u origin master
```

Now, your jekyll app should be immediately viewable at `http://USERNAME.github.com` after installation has finished on the servery behind GitHub Pages. But we are not going to use this GitHub Pages for hosting. I will discuss why on the next part.

## Hosting your blog

There are generally three hosting options:

1. Hosting on [GitHub Pages][ghpage] (free)
2. Hosting on your own VPS server (not free, like AWS, Heroku or Digital Ocean)
3. Use static site hosting service like [netlify][netlify] (free and **recommended in this tutorial**)

GitHub Pages is actually powered by jekyll behind the scene, which means you can host your jekyll app entirely on GitHub Pages for free. But this approach has some limitations. For one, it lacks the ability to use customized gem-based theme like what's used in this tutorial. Hosting on your server generally means more flexibility but you have to pay for the server itself and all the maintenance work is on you. Netlify is an awesome service for hosting personal static blog and offers features like custom domain and HTTPS, which I will cover later.

### Hosting on Netlify

Create an account on `https://www.netlify.com/` and link with your GitHub:

![Netlify Create New App](/assets/images/netlify-create-new-site.png)

Create a new site and choose the repository `USERNAME.github.com` from the list:

![Netlify Choose Repo](/assets/images/netlify-choose-repo.png)

Select the branch where the site should be generated from, usually it's `master` or `gh-pages`. Change the `Build command` and `Public directory` if you need to. (default value here is fine)

![Netlify deploy site](/assets/images/netlify-deploy-site.png)

Now you site is hosted on Netlify. However in order to use your custom domain, we need to change the DNS setting. Go to your **domain registar** (e.g. GoDaddy) and change the name servers to use netlify's name servers. This will take some time to take effect, depending on your previous DNS cache.

![Netlify dns site](/assets/images/netlify-dns.png)

You can verify whether NS change has taken effect:

```bash
$ dig sfdye.com NS +noall +answer

; <<>> DiG 9.9.7-P3 <<>> sfdye.com NS +noall +answer
;; global options: +cmd
sfdye.com.		3600	IN	NS	dns1.p02.nsone.net.
sfdye.com.		3600	IN	NS	dns2.p02.nsone.net.
sfdye.com.		3600	IN	NS	dns3.p02.nsone.net.
sfdye.com.		3600	IN	NS	dns4.p02.nsone.net.
```

Once Netlify is managing your domain name, go to the `SSL/TLS certificate` setting and enable HTTPS. Behind the scene, it's using [Let's Encrypt][letsencrypt] which is a free SSL certificate provider.

![Netlify SSL](/assets/images/netlify-ssl.png)

Voilà, that's all! You have your own static generated blog hosted for free, version controlled on GitHub and secured by HTTPS.

## CMS (Optional)

Jekyll is awesome but still it lacks some capabilities compared to full-fledged blogging frameworks like `Wordpress`. One of the pain points I encountered before is that there is no GUI editor or more user-friendly interface to edit posts. Now we are in luck.

### [jekyll-admin][jekyll-admin]

This is a nice project born in one edition of `Google Summer of Code`, which provides a much user-friendly way to manage pages, posts and other resources. Installation is very simple, you can check out their GitHub page for more information.

![jekyll admin](/assets/images/jekyll-admin.png)

### Hosted CMS for static site

Jekyll-admin is only meant for local development. When you deploy your site for GitHub Pages or Netlify, the `/admin` won't be available because remember, it is a static site after all.

There is hosted solution [siteleaf][siteleaf]. It's officially recommended by jekyll-admin. I suggest you to check it out. Note that there are some known issues like siteleaf will strip all comments in the `_config.yml`.

_With that, thanks for reading. Leave a comment if you have any questions._

### Update (2018-02-21)

If you are getting annoying emails from GitHub, which does not provide a unsubscribe button.

> You are attempting to use a Jekyll theme, "minimal-mistakes-jekyll", which is not supported by GitHub Pages.

You can disable this by deleting both the `master` and `gh-page` branch and create a new branch called `deploy`, then on netlify assign `deploy` as the production deploy branch.

[jekyll]: https://jekyllrb.com
[netlify]: https://www.netlify.com/
[rbenv]: https://github.com/rbenv/rbenv
[rvm]: https://rvm.io/
[ghpage]: https://pages.github.com/
[mmistakes]: https://mmistakes.github.io/minimal-mistakes/
[minima]: https://github.com/jekyll/minima
[mmistakes-guide]: https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#starting-from-jekyll-new

[index.html]: https://github.com/mmistakes/minimal-mistakes/blob/master/index.html]
[staticgen]: https://davidwalsh.name/introduction-static-site-generators
[letsencrypt]: https://letsencrypt.org/
[jekyll-admin]: https://github.com/jekyll/jekyll-admin
[siteleaf]: https://www.siteleaf.com/
