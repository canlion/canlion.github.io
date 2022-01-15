---
title: "사이드바에 카테고리 생성"
# folder: "blog"
excerpt: "카테고리별로 포스트 리스트를 볼 수 있도록 카테고리 목록 생성하기"

categories:
  - blog
tags:
  - [blog]

toc: true
toc_sticky: true

date: 2022-01-16 00:33
last_modified_at: 2021-01-16 00:33
---

## 사이드바에 카테고리 목록 생성하기

![카테고리목록예시](/assets/images/post/220116/blog_add_category_0.png){: .align-center}

**참고자료: [공부하는식빵맘님의 평생공부블로그](https://ansohxxn.github.io/blog/category/#-%EC%84%9C%EB%A1%A0)**

### 포스트에 카테고리 추가
```ruby
---
title: "사이드바에 카테고리 생성"
excerpt: "카테고리별로 포스트 리스트를 볼 수 있도록 카테고리 목록 생성하기"

categories:
  - blog  # 카테고리 추가

...

---
```

### 카테고리별로 포스트 목록 페이지 생성
1. `_pages/categories` 디렉토리 생성
2. 생성한 디렉토리에 카테고리별 포스트 목록을 생성하는 페이지 생성
    {%raw%}
    ```ruby
    ---
    title: "blog"  # 페이지 상단에 표시되는 타이틀 
    layout: archive
    permalink: categories/blog  # 페이지 경로 🌟
    author_profile: true
    sidebar_main: true
    ---

    {% assign posts = site.categories.blog %}  # 포스트 카테고리
    {% for post in posts %}
      {% include archive-single.html type=page.entries_layout %}
    {% endfor %}
    ```
    {%endraw%}

### 사이드바 추가
`_data/navigation.yml`에 카테고리 목록을 작성
```yaml
sidebar_category:  # 💥
  - title: ML - Machine Learning
  - title: ML - Deep Learning  # 상위 카테고리
    children:
      - title: "tensorflow"  # 하위 카테고리
        url: categories/tensorflow  # 카테고리별 포스트 목록 생성 페이지 경로 🌟
      - title: "pytorch"
        url: categories/pytorch
      - title: "boostcamp"
        url: categories/boostcamp
  - title: etc..
    children:
      - title: "blog"
        url: categories/blog
```

`_config.yml`에 포스트와 페이지에서 사이드바를 띄우도록 설정
```yaml
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      #########################
      author_profile: true
      sidebar:
        nav: "sidebar_category"  # 💥
      #########################
      ...
      
  - scope:
      path: ""
      type: pages
    values:
      #########################
      author_profile: true
      sidebar:
        nav: "sidebar_category"  # 💥
      #########################
```

### 카테고리마다 포스트의 수 표시
`_includes/nav_list` 수정

{% raw %}
```ruby
...
# line: 18
{% for child in nav.children %}
  <li>
    # navigation.yml에 정의한 카테고리명 출력
    <a href="{{ child.url | relative_url }}"{% if child.url == page.url %} class="active"{% endif %}>{{ child.title }}</a>
    {% for category in site.categories %}  # 존재하는 포스트 카테고리 순회
      # 카테고리 이름이 일치하면 카테고리 포스트 수 출력
      {% if child.title == category[0] %}
        ({{category[1].size}})
      {% endif %}
    {% endfor %}
  </li>
{% endfor %}
...
```
{% endraw %}