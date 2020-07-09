---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: "{{ .Date }}"
author: Sharleen Weatherley
summary: "This is the summary that will appear on the main page."
draft: false

tags:
- tag1
- tag2

coverImage: http://res.publicdomainfiles.com/pdf_view/145/13978843819386.jpg
coverCaption: "A short caption (Source: The source)"
thumbnailImage: http://res.publicdomainfiles.com/pdf_view/145/13978843819386.jpg
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: partial

blogdown::html_page:
  toc: no
---

```{r setup, echo = FALSE, warning = FALSE, message = FALSE}

knitr::opts_chunk$set(
  comment = "",
  warning = FALSE,
  echo = TRUE,
  message = FALSE
)

library(emo)  # for emojis
library(devtools) # for session_info()

# `r emo::ji("smile")`  # Function call for emojis
```

<hr>

# Session Info

```{r reproducibility, echo = FALSE}

session_info()

```