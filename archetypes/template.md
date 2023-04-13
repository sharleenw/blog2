---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: "{{ .Date }}"
author: Sharleen Weatherley
summary: "This is the summary that will appear on the main page."
description: "This is the description that will appear on Twitter."
draft: false

tags:
- tag1
- tag2

twitterImg: https://res.publicdomainfiles.com/pdf_view/145/13978843819386.jpg
coverImage: https://res.publicdomainfiles.com/pdf_view/145/13978843819386.jpg
thumbnailImage: https://res.publicdomainfiles.com/pdf_view/145/13978843819386.jpg

coverCaption: "A short caption (Source: Source)"
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: partial


---

```{r setup, echo = FALSE, warning = FALSE, message = FALSE}

knitr::opts_chunk$set(
  comment = "",
  warning = FALSE,
  echo = TRUE,
  message = FALSE
)

library(devtools) # for session_info()

```

<hr>

# Session info

```{r reproducibility, echo = FALSE}

devtools::session_info()

```
