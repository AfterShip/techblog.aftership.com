# techblog.aftership.com

tech blog for AfterShip

## global package requried:

``` shell
npm i hexo -g
```

## to generate a post:

``` shell
hexo new post xxxx # xxxx is the title of article

# following files is generated:
source/
  |- _posts/xxxx.md # <-- the md file will be edited
  |- _posts/xxxx/   # <-- the asset folder
```

in the post, if you wanted to include an image,

1. put it in the \_posts/xxxx.md folder
2. in the .md file, use `{% asset_img "spaced asset.jpg" "spaced title" %}` to include the image

@see [hexo-asset-folders](https://hexo.io/docs/asset-folders.html)

## front matters

front matters tells hexo to display some meta data such as `author`, `title` `tags` of the page
this could vastly help seo and search-ability.

This block ultimately is compiled as a `.yml`

```
---
title: article of the article
author: John Doe (optional)
date: 2016-12-19 16:45
categories: # (optional)
  - category 1
  - category 2
  - category 3
tags: # (optional)
  - tag1
  - tag2
---
```

## article summary

use html `<!-- more -->` to indicate that the the above is the summary of the article

```
---
# frontmatter here
---

// this is the summary of an article displayed in the list page

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,

<!-- more -->
```

## danger block

``` html
<div class="tip">
  here is a handy tip
</div>
```

## deploy

``` bash
hexo generate --deploy
```
this will generate all of the static files and deploy them to the `gh-pages` branch on github

## why hexo?
hexo is a static site generator talored for github, it has a realtively low learning curve compared to github recommended `jekyll` which uses `ruby` + `liquid`.

@see [hexo](https://hexo.io/docs)
