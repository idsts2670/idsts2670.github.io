---
layout: post
title: Kaggle AMEX competition EDA sample
date: 2024-01-16 00:00:00-0400
description: Introduction of Amex competition EDA jupyternotebook
tags: jupyter
categories: sample-posts
giscus_comments: false
related_posts: false
---

{::nomarkdown}
{% assign jupyter_path = "assets/jupyter/amex-eda.ipynb" | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/amex-eda.ipynb %}{% endcapture %}
{% if notebook_exists == "true" %}
    {% jupyter_notebook jupyter_path %}
{% else %}
    <p>Sorry, the notebook you are looking for does not exist.</p>
{% endif %}
{:/nomarkdown}
