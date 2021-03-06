---
title: "Migrating Wordpress to Gridsome on Netlify through GitHub Actions"
slug: migrating-wordpress-to-gridsome-on-netlify-through-github-actions
description: "This post discusses the whole process that migrates existing Wordpress websites to gridsome based on vue.js, deploys them to Netlify through GitHub Actions."
date: "2020-01-03"
author: Justin-Yoo
tags:
- front-end-web-dev
- wordpress
- gridsome
- netlify
- github-actions
- migration
cover: https://sa0blogs.blob.core.windows.net/devkimchi/2020/01/migrating-wordpress-to-gridsome-on-netlify-through-github-actions-00.png
fullscreen: true
---

This post discusses the whole process that migrates existing [Wordpress][blog wordpress] websites to [gridsome][blog gridsome] based on [vue.js][vuejs], deploys them to [Netlify][netlify] through [GitHub Actions][gh actions].

## Why Migration? ##

There's no doubt that [Wordpress][blog wordpress] is one of the most well-known and well-made blog publishing tools in the world. However, I have to deploy it to somewhere and maintain it. It's OK unless any issue is coming up, which make our lives harder. [Wordpress][blog wordpress] has continuously evolved so that it automatically updates its core, plugins and themes, but it sometimes stops working without knowing it. We also want to avoid our maintenance efforts as much as possible, including website and database. On the other hands, hosted Wordpress is really handy because we don't have to care for most things. But it becomes costly if we need to add custom domains, SSL certificates, custom plugins and custom themes.

Because of these issues, I've been looking for alternatives using static site generators such as [Jekyll][blog jekyll], [Hexo][blog hexo] and [Hugo][blog hugo], but none of them was satisfying. Then I found [Gatsby][blog gatsby] based on [React][reactjs]. As I've never used React before, although it looks very promising, I didn't even start taking a look. Instead, I looked for another tool based on [vue.js][vuejs] that I used to build some web apps before, and ola! found [gridsome][blog gridsome]! So, I played around this tool and wrote [this blog post][blog post 1] and [this post][blog post 2]. And finally, I decided to use [gridsome][blog gridsome] for my migration.


## Why GitHub Actions? ##

Instead of [Azure DevOps][az devops], I chose [GitHub Actions][gh actions] because I don't have to leave GitHub to build and publish. If I select [Azure DevOps][az devops], I have to create an instance, followed by creating a project and configuring it. Too much work for my simple blog hosting. Additionally, the fact that [GitHub Actions is a fork of Azure Pipelines][gh actions hosted runners] is appealing to me, and it's modified easier for us to use. And the runners for [GitHub Actions][gh actions] are running on Azure. Why not using it then?

> If there is no [GitHub Actions][gh actions], I would definitely go for [Azure DevOps][az devops], because it gives the best user experience as a serviced and free service supporting open source projects.


## Why Netlify? ##

[Azure Blob Storage][az blob storage] for [static website hosting feature][az blob static website 1] was my first consideration in the first place. However, [one blob storage can host only one static website][az blob static website 2]. I'm hosting three websites &ndash; [justinchronicles.net][jc], [aliencube.org][ac] and [devkimchi.com][dk]. One blob storage for three websites is not possible. In addition to this, after the [custom domain configuration][az storage custom domain], to [use HTTPS connection][az storage https], I have to use [Azure CDN][az cdn] or something similar extra work is required, which I wanted to avoid. Therefore, [Netlify][netlify] has become my choice.


## Converting Wordpress Posts ##

### XML Export ###

[Wordpress][blog wordpress] doesn't support markdown export out-of-the-box. There are third-party plugins for it, but not working as my expectations. Therefore, I just exported all posts using the built-in [export feature][wp export]. I've now got the XML file, and I needed it to be converted to markdown.


### Markdown Conversion ###

This took most of my migration time and efforts. First of all, I used [wordpress-export-to-markdown][wp2md] to convert the XML file to markdown files. But the result markdown files didn't have enough [front-matter][frontmatter] details, so I had to open every sing file to update each front-matter. As [Jekyll][blog jekyll]'s front-matter has been widely adopted to other static site generators, including [gridsome][blog gridsome], it wasn't that difficult. It was just a matter of time for conversion.

The front-matter after the markdown conversion from XML looks like this:

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=frontmatter-1-en.yaml

But the default front-matter generated by [gridsome][blog gridsome] has more attributes:

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=frontmatter-2-en.yaml&highlights=3-4,6-9

Therefore, except `title` and `date`, every other attribute were manually updated. As there were a relatively small number of posts, it didn't take too much, but I wouldn't do it again. Anyway, I completed all the markdown conversion. Yay! Nearly there.


## Preparing Gridsome Blog Theme ##

[gridsome][blog gridsome] offers many official and third-party [starters][gs starter]. I chose [Bleda][gs starter bleda] because it basically includes many other plugins out-of-the-box.


## Preparing Gridsome Plugins ##

As mentioned earlier, the starter contains almost everything I needed, except...


### gridsome-plugin-remark-embed ###

This plugin is to [embed social media][gs plugin embed] for Twitter, YouTube and GitHub gist. However, as it has a bug on the gist embedding script, I raised a [PR][gs plugin embed pr] to fix the bug and have been waiting for the official release. Instead, while I'm waiting for the official release, I locally copied the updated script so that it can be used later on in the GitHub Actions.


### vue-disqus ###

This plugin is to [add comment feature][gs plugin comments]. I've already been using [Disqus][disqus] for a while and reviewing whether I keep using it or not after the migration. But as I've already got many valuable comments there, I decided to keep it for now.


## Configuring Gridsome ##

### Metadata Configuration ###

All metadata, including the favicon, were updated for my blog settings.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=config-metadata-en.js


### Permalink Configuration ###

To keep the existing Wordpress permalink structure, `gridsome.config.js` updated the permalink settings.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=config-permalink-en.js&highlights=3-4

Also, the RSS feed URL has been updated.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=config-rss-feed-en.js&highlights=9-10


### GitHub Gist Style Configuration ###

The following stylesheet has been added to `src/main.js` to follow the GitHub gist style.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=main-gist.js


### Cover Image Configuration ###

The original starter doesn't show the cover images on the list page. I changed `/src/components/PostItem.vue` to display the cover images.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=post-item-cover.vue&highlights=3-5

And this also needs the GraphQL query updated. Both `/src/templates/Author.vue` and `/src/templates/Tag.vue` have been updated.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=graphql.vue&highlights=13

All the settings on gridsome have been completed.


## Preparing Netlify Instance ##

We need an instance on [Netlify][netlify] for static website hosting. We can directly integrate GitHub with Netlify for it, but we're going to use [GitHub Actions][gh actions]. Therefore, we create an instance manually, by dropping any file.

![][image-01]

Now, the newly created instance has its own instance ID and site name, which will be used in GitHub Actions.

![][image-02]

Finally, we need to create a [Personal Access Token][netlify pat] for deployment through Netlify CLI.


## Preparing GitHub Actions Workflow ##

We've now got the static website artifact and Netlify instance. The GitHub Actions workflow will orchestrate all the things.

### Event ###

I'm using the `master` branch to update the original starter, and the `dev` branch to publish posts. Therefore, I set up the push event only triggered by the `dev` branch.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-event.yaml&highlights=2,4

### Runner ###

I used Ubuntu LTS 18.04 as the GitHub-hosted runner.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-runner.yaml&highlights=3

### Steps ###

The first step should be the checkout action.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-step-checkout.yaml&highlights=3

All the rest actions are using the built-in shell action. This action downloads and installs the Netlify CLI. Due to the `-g` option, we should use the `sudo` command here.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-step-install-netlify.yaml&highlights=4

This action restores all the npm packages.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-step-restore-npm-packages.yaml&highlights=4

This interim action will replace `Gist.js` until the official release is updated.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-step-monkey-patch.yaml&highlights=4

This action builds the app.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-step-build-app.yaml&highlights=4

As the second last step, it copies the domain redirection settings to `dist`.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-step-copy-redirects.yaml&highlights=4

And finally, the `dist` directory is published to Netlify through CLI. Both `NETLIFY_SITE_ID` and `NETLIFY_AUTH_TOKEN` are secrets stored in the repository settings.

https://gist.github.com/justinyoo/53cefa22732c8bc33348aa99e0674a37?file=github-actions-step-publish-app.yaml&highlights=4

We all complete the blog migration from Wordpress to gridsome. Now, I don't have to worry about maintenance unless GitHub shuts down.


[image-01]: https://sa0blogs.blob.core.windows.net/devkimchi/2020/01/migrating-wordpress-to-gridsome-on-netlify-through-github-actions-01.png
[image-02]: https://sa0blogs.blob.core.windows.net/devkimchi/2020/01/migrating-wordpress-to-gridsome-on-netlify-through-github-actions-02.png

[jc]: https://justinchronicles.net
[ac]: https://blog.aliencube.org
[dk]: https://devkimchi.com
[reactjs]: https://reactjs.org/
[vuejs]: https://vuejs.org/
[netlify]: https://www.netlify.com/
[disqus]: https://disqus.com/

[blog wordpress]: https://wordpress.org/
[blog jekyll]: https://jekyllrb.com/
[blog hexo]: https://hexo.io/
[blog hugo]: https://gohugo.io/
[blog gatsby]: https://www.gatsbyjs.org/
[blog gridsome]: https://gridsome.org/

[blog post 1]: https://devkimchi.com/2019/12/13/publishing-static-website-to-azure-blob-storage-via-github-actions/
[blog post 2]: https://devkimchi.com/2019/12/18/building-ci-cd-pipelines-with-github-actions/

[az devops]: https://azure.microsoft.com/services/devops/?WT.mc_id=devkimchicom-blog-juyoo
[az blob storage]: https://docs.microsoft.com/azure/storage/blobs/storage-blobs-overview?WT.mc_id=devkimchicom-blog-juyoo
[az blob static website 1]: https://docs.microsoft.com/azure/storage/blobs/storage-blob-static-website-how-to?tabs=azure-portal&WT.mc_id=devkimchicom-blog-juyoo
[az blob static website 2]: https://docs.microsoft.com/azure/storage/blobs/storage-blob-static-website?WT.mc_id=devkimchicom-blog-juyoo
[az storage custom domain]: https://docs.microsoft.com/azure/storage/blobs/storage-custom-domain-name?WT.mc_id=devkimchicom-blog-juyoo
[az storage https]: https://docs.microsoft.com/azure/storage/blobs/storage-https-custom-domain-cdn?WT.mc_id=devkimchicom-blog-juyoo
[az cdn]: https://docs.microsoft.com/azure/cdn/cdn-overview?WT.mc_id=devkimchicom-blog-juyoo

[gh actions]: https://github.com/features/actions
[gh actions hosted runners]: https://help.github.com/en/actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners#about-github-hosted-runners

[wp export]: https://wordpress.org/support/article/tools-export-screen/
[frontmatter]: https://jekyllrb.com/docs/front-matter/
[wp2md]: https://github.com/lonekorean/wordpress-export-to-markdown

[gs starter]: https://gridsome.org/starters/
[gs starter bleda]: https://gridsome.org/starters/bleda/
[gs plugin embed]: https://gridsome.org/plugins/@noxify/gridsome-plugin-remark-embed
[gs plugin embed pr]: https://github.com/noxify/gridsome-plugin-remark-embed/pull/33
[gs plugin comments]: https://gridsome.org/docs/guide-comments/

[google webfonts]: https://fonts.google.com/
[google webfonts nanumgothic]: https://fonts.google.com/specimen/Nanum+Gothic

[netlify pat]: https://docs.netlify.com/cli/get-started/#obtain-a-token-in-the-netlify-ui
