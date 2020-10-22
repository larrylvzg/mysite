---
author: "larrylvzg"
title: "Leaf page bundle with page resources"
date: 2020-10-20T12:00:06+09:00
description: "Testing for the leaf bundle page resources"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
authorEmoji: 👻
tags: 
- hugo
- shortcode
---


In old days, when you want to put a image in your page, you have to place this image into the **static** folder in your hugo project. It will be hard to distinguish as the quantity of your pages and images increasing.

Hugo page bundle is mainly used for the content organization. With this feature, You can place the page resources like images, documents(pdf, doc...) together with content(markdown file) itself, so your contents would be better for maintenence.

## The display of result for this shortcode
{{< pagebundle/img "**.jpg" "Those cupcakes are way overcooked!" >}}

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

