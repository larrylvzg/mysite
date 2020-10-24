---
author: "larrylvzg"
title: "Page Bundle Image References"
date: 2020-10-20T12:00:06+09:00
description: "Regarding to how to reference images in the page bundle"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
authorEmoji: 👻
tags: 
- hugo
- shortcode
---


In previous version of `Hugo`, when you want to reference a image file on your page, you have to place this image file into the **static** folder of your hugo project. It would be difficult to distinguish over time.

`Page Bundle` is mainly used for the content organization. With this feature, You can place the page resources like images, documents(pdf, doc...) together with content file(.md, .html etc) itself, so your content structure would be more intuitive.

## The display of result for this shortcode
{{< pagebundle/img "images/deer.jpg" "A beautiful deer!" >}}

## How to organize your content with images.
#### Directory structure:
```
└── posts
    ├── _index.md
    ├── my-test-page
    │   ├── images
    │   │   └── deer.jpg
    │   └── index.md
```
#### Shortcode
Directory structure:
```
../hugo-theme-zzo/layouts/shortcodes/
├── pagebundle
│   └── img.html
```

Content of img.html:
```
{{ $img := $.Page.Resources.GetMatch (.Get 0)}}
{{ $img_caption := .Get 1 | safeHTML }}
<figure>
	<img src="{{ $img.RelPermalink }}" alt="{{ $img_caption }}" />
	<figcaption style="text-align: center;">
		<strong>{{ $img_caption }}</strong>
	</figcaption>
</figure>
```

