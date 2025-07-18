---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
lastmod: {{ .Date }}
categories:
  - "{{ dateFormat "2006/01" .Date }}"

draft: true

slug: "{{ dateFormat "20060102" .Date }}"
thumbnail: ""
tags:
  - ""
summary: ""
---

<!--more-->
