---
draft: true
date: '{{ .Date }}'
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
description: ""
tags: ["post"]
searchHidden: false
disableShare: false
canonicalURL: 'https://jhlee7854.github.io/posts/{{ .File.ContentBaseName }}'
ShowCanonicalLink: true
CanonicalLinkText: "Originally published at"
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true
---
