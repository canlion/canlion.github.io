---
title: "http"
layout: archive
# layout: categories
permalink: categories/http
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories.http %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
