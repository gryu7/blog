---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
categories: 
  - "{{ dateFormat "2006/01" .Date }}"

draft: true

slug: ""
thumbnail: ""
tags:
  - ""
---

