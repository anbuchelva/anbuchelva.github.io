---
title: Forestry.io as CMS for Hexo
categories:
  - blog
tags:
  - blog
  - hexo
  - cms
keywords:
  - forestry
  - netlify
  - hexo
comments: true
draft: false
toc: true
date: 2019-12-23T19:37:00.000+00:00
updated: 2019-12-25T12:22:00.000+00:00
thumbnailimage: 'https://res-4.cloudinary.com/crunchbase-production/image/upload/c_lpad,h_250,f_auto,b_white,q_auto:eco/rkt2tgioo2njucinkhmd'
---

There are many ways to write and publish blog posts, when it comes to git backed static websites. I started with text editor for writing, terminal for building, then push it to [Github pages](https://pages.github.com/) for publishing. Then I started using [Netlify](https://app.netlify.com) for building static pages, but was still using text editor and terminal locally.

<!--more-->

One fine day, I set up Netlify CMS for content management, as I wanted it to write in any PC, which do not have git & npm installed. I tried [forestry.io](https://forestry.io) for writing posts for Jekyll & Hugo (during my initial testing to decide which static site platform to choose). But I couldn't make it work for [Hexo](https://hexo.io) during the initial times.

Now I have succeeded using forestry.io as CMS for Hexo and this post is being written in the same platform and sharing my experiences here. Thanks for the help from [Frank Taillandier](https://github.com/DirtyF).

{{< alert info >}}

There is a post that explains how to [Integrate Netlify CMS with Hexo]({{< relref "integrate-netlify-cms-with-hexo" >}}). You may refer it, which will be useful to understand the CMS on hexo platform.

{{< /alert >}}

### Steps to Connect Hexo

Login to forestry.io, then click add site button. There is no dedicated option available to connect to Hexo, use the other option.

![forestry select platform](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1577124196/images/Screenshot_89_ybvaxy.png)

Then select your git based repository provider, where your blog sources are located, keep the Quick Setup via OAuth checked.

![forestry select git provider](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1577124603/images/Screenshot_90_gixnmd.png)

Select your repository and the right branch.

![forestry select right branch](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1577124801/images/Screenshot_91_ogb0bu.png)

### Configure Sidebar

Once imported, you will get a wizard to configure sidebar; by doing it forestry creates a folder in the git repo named `.forestry`, where all the configuration files are stored.

![forestry configure sidebar](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1577125841/images/Screenshot_93_sj7txo.png)

You can refer this blog's config [here](https://github.com/anbuchelva/blog/tree/source/.forestry). You can configure the sidebar manually or you can just take those files from my repo and place it in yours.

### Handle Media

You can connect to any of the following sources to source your images.

1. Repository, where the blog is being sourced
2. Cloudinary
3. Amazon S3
4. Netlify Large Media

![forestry handle media](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1577126114/images/Screenshot_95_ndprsb.png)

There are two paths/folders to be mentioned, where the file to be uploaded and the public path where the media files would be sourced. This blog is configured with [cloudinary](https://cloudinary.com/) free version.

### Front Matter

This is the place where the front matter are configured. It can be imported from the existing blog posts or can be configured manually. This creates a folder in the following path of the git repo `.forestry/front_matter/templates`.

![forestry front matter example](https://res.cloudinary.com/anbuchelva/image/upload/f_auto,q_auto/v1577126583/images/Screenshot_98_evjy1i.png)

### Preview Builds

One of the best things that I like from forestry is real time preview builds. The site is being built from the content that is being written, even before committing to git repo. The build time takes \~10 seconds to render the preview (the time is for my blog, your mileage may vary), which is 10 times lesser than netlify auto build.

Since Hexo is not officially supported by forestry, we have to configure the preview commands in two places.

- The following code to be added in the package.json file. Example [here](https://github.com/anbuchelva/blog/blob/source/package.json).

```
"scripts": {
"preview": "hexo server --draft --port 8080",
},
```

- The preview command that we added, to be called in .forestry/settings.yml. Example [here](https://github.com/anbuchelva/blog/blob/source/.forestry/settings.yml).

```
build:
preview_output_directory: public
install_dependencies_command: npm install
preview_docker_image: node:10
mount_path: "/srv"
working_dir: "/srv"
instant_preview_command: npm run preview
```

### Deployment

forestry stopped supporting deployment for sometime :pensive:. So, have to use some other services for ci & cd, such as [Netlify](https://app.netlify.com) or [Travis-CI](https://travis-ci.org/).

### How it works?

Forestry doesn't do any magic on git. It just takes the repo & branch which was configured at the beginning and commit the changes directly into the Repository Branch.

Each time you hit save button, while writing a blog post it commits to git repo. Since, Hexo is not officially supported by forestry, there is no inbuilt ON/OFF buttons available to stop draft posts getting published.

It is bit annoying, as my site is configured to build automatically for each commit and I don't want to display half written posts to my visitors. (yes! there is an option available in netlify build settings to stop auto deploying builds - which I don't want to modify for each builds!)

I feel netlify-cms was done well in this aspect, which creates a pull request and it merge the PR, only when I hit publish.

### Handling Draft for Hexo

If you are using Hexo for sometime, you would know how the drafts are handled. Detailed explanation [here](https://hexo.io/docs/writing.html#Drafts). In short, the draft posts are available in `_drafts` folder and published posts are available in `_posts` folder. Moving a post from \_drafts to \_posts is not possible using forestry method.

To overcome this issue, I made few changes in front matter for getting the author's input, whether the post is to be considered as draft - ON or OFF and the build script.

When the draft is set to OFF, the script moves the post from \_drafts to \_posts folder. However, it doesn't commit anything back to the source repo. The post still be available under \_drafts.

my shell script is [here](https://github.com/anbuchelva/blog/blob/source/.forestry/draft.sh) (i'm not an expert in writing shell)

```
for file in source/_drafts/_.md; do
    if head -20 "$file" | grep -Fiqx "draft: false"
        then
            mv "$file" "source/_posts/"
            echo "Publishing $(basename "$file") from draft"
        else
            echo "Keeping $(basename "$file") in draft"
    fi
done
for file in source/_posts/_.md; do
    if head -20 "$file" | grep -Fiqx "draft: true"
        then
            mv "$file" "source/_drafts/"
            echo "Revoke publish post $(basename "$file") to draft"
    fi
done
echo "Clean database"
hexo clean
echo "Build Posts"
hexo generate
```

I call this script via `package.json` and `netlify.toml` file, which builds my site automatically.

### Next Steps

I would like to add all the snippets, which are used in Hexo Next theme, so that I can call them easily whenever I write posts.

### Next steps from Forestry, which I would expect

1. I wish they support Hexo officially :wink:
2. Start supporting tables in wysiwyg editor
3. Fix small bugs in handling codes in the editor.

### What is the difference compared to Netlify CMS?

1. Netlify CMS requires many steps like configuring admin folder, enable identity services, give yourself permission, etc., which would be difficult for some people. I feel configuring and using forestry is easy.
2. The preview builds are faster than Netlify.
3. Forestry can be used in mobile phones, whereas Netlify CMS works only on desktop computers.
4. Need not to build admin panel with site in Forestry, whereas it is mandatory in Netlify CMS. Removing `<head>` and `<body>` codes of Netlify CMS, increases Google Page speed slightly.
5. The only downside I see from Forestry is that it commits the changes directly to git repo, whereas Netlify CMS creates a pull request (if github is used as git provider) for each post. This is best way to maintain draft posts.

Thanks for reading this post.  
Wishing everyone Merry Christmas :christmas_tree: :santa: and a Happy New Year 2020! :tada:

{{< alert info >}}

Please make a PR, if you feel that this can be done in a better way. I am happy to make it easy for everyone.

{{< /alert >}}
