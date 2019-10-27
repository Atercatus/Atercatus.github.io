---
layout: single
header:
  overlay_image: /assets/images/random-cover/1.png
  teaser: /assets/images/random-cover/1.png
  caption: Photo by Todd Quackenbush on Unsplash

title: "Minimal mistakes collection 적용하기"
excerpt: "collection 을 적용해보자"

categories:
  - github pages
  - collection
tags:
  - github pages
  - collection

last_modified_at: 2019-04-13T08:06:00
---

## Minimal Mistake 에서 Collection 을 이용해보자

[Link](https://talk.jekyllrb.com/t/collections-setting-front-matter-defaults-with-nested-directories/1643/16?u=atercatus)

- 아래의 글은 위 링크에 질문자가 recap을 작성한 것을 옮겨놓은 것입니다.

---

I have everything working as I want it now. Here’s a recap for anyone following, or you future folk who come in search of answers about config.yml, collections, collections_dir, and front-matter defaults in collections.

directory and file structure, from site root:

```
docs/
    _manual-name
        content.md
    _tutorial-name
        content.md
```

Collections Directories — Important take aways:

1. Collections Directories must not have an underscore at the beginning of its name.
2. Collections Directories will not render in the URL path unless you manually put them into the permalink keypair.
3. Subdirectories of Collections Directories must be prefixed with an underscore (assuming they are each a collection themselves).

In config.yml

```
# Collections
collections_dir: docs
collections:
  manual:
    permalink: "/:collection/:path/"
    output: "true"
  tutorial:
    permalink: "/:collection/:path/"
    output: "true"
```

Collections Config - Important take aways:

1. value for collections_dir must not start with an underscore (because Collections Directories cannot, and this value must match what your directory is named exactly.)
2. collections value must not start with an underscore, despite the fact that their actual directory must be prefixed with an underscore. (This point is super confusing for some of us.)
3. If you want the collections_dir to show in the URL, you need to manually prefix it to the permalink value. It does not need a slash before it (I think!)

```
# defaults:
      - scope:
          path: ""
          type: "manual"
        values:
          layout: "single"
          author_profile: false
          read_time: true
          share: false
          related: false
          sidebar:
           nav: "manual"
          toc: true
      - scope:
          path: ""
          type: "tutorial-name"
        values:
          layout: "single"
          author_profile: false
          read_time: true
          share: false
          related: false
          sidebar:
           nav: "manual"
          toc: true
```

Front-matter Defaults Important Take aways

1. Stay away from path unless you need to be specific. I had a lot of trouble trying to get it to do anything useful.
2. You need a - scope: entry for each collection within the collections_dir. It must not have an underscore at the beginning (despite the fact the actual collection requiring an underscore as the first character of the directory name)

I tried to make one defaults entry that covered all collections within the collections_dir, because mine all happen to have the same front-matter requirements. I thought I could do this by using a glob with the path: like path: docs/\* but that doesn’t work.

Happy to receive clarifications or corrections to this. I think some of this should find its way back into the official docs, but I’m too newbie with Jekyll to feel comfortable with that. However, if someone else is comfortable with that, I’m happy to review or contribute to the effort. Just reach out. Thanks again to anyone who wrote in and offered assistance.
