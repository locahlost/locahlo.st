# Locahlo.st

This repository is locahlo.st website source.

Website is static and is build using [Hugo](https://gohugo.io/).
Currently, used theme is [beautifulhugo](https://github.com/halogenica/beautifulhugo/).

## Advices to write a post

1. Create a branch to work on. You can name it like your post.
2. Your post should be in organized directories. `content/post/YEAR/MONTH/post_name/`
3. Post file is named `index.md` to allow web browsers to find in easily once website is build.
4. Any post attachement should be in your post directory.
5. Post `index.md` must have a header which contain metadata. More info bellow.
6. Commit with obvious name like `[post][fix] awful typo` or `[post][new] The post that rocks!`
7. If you want a peer verification, push your branch and ask for review :-)
8. Since website build and publication isn't automatized, a pull request should be done rather than pushing to master.

```
---
title: The post that rocks!
subtitle: The secret to cool post is yet known
author: Charlie Root
date: 2020-03-20
draft: true
tags:
  - tag 1
  - second tag
  - 3st tag
---

Here you can write an introduction that will be shown in small post cards.
Everything following tag won't be shown and will show a [more].

<!--more-->

Not shown, lorem ipsum dolor sit amet.
```

## License

Website and sources are under Attribution-ShareAlike 4.0 International.
Details about license can be found in LICENSE file at repository source root.
