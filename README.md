# techblog.aftership.com

tech blog from AfterShip

## global package:
``` shell
npm i hexo -g
```

## to generate a post:

``` shell
hexo new post xxxx # xxxx is the title of article
```

## front matters

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
Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,

<!-- more -->
```

## danger block

``` html
<div class="tip">
  here is a handy tip
</div>
```

## deploy (coming soon)

TODO: write doc about how to deploy
