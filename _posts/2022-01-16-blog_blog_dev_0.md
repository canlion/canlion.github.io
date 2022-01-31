---
title: "블로그 꾸미기 - 0"
excerpt: "링크 색상 변경 / 글 목록의 작성날짜, 설명 한줄에 출력"

categories:
  - blog
tags:
  - [blog]

toc: true
toc_sticky: true

date: 2022-01-16 15:02
last_modified_at: 2022-01-16 15:02
---

### TODO
* 블로그 하이퍼링크텍스트 색상 변경
* 글 목록의 작성날짜, 글 설명을 한줄에 출력

|**Before**|**After**|
|-------|------|
| ![수정전](/assets/images/post/220116/blog_deco_0_before.png) | ![수정후](/assets/images/post/220116/blog_deco_0_after.png) |

### 하이퍼링크 텍스트 색상 변경
 로컬에 지킬 서버를 띄워서 개발자 도구로 글 목록의 글 제목을 찍어보면 `_reset.scss`에 `a`태그의 색상을 지정한다. `_reset.scss`에서 찾아보면 `$link-color`가 컬러값으로 지정되어있다.

 `_variables.scss`에서 `$link-color`를 수정해도 제목 색상 수정이 안된다. 생각해보니 contrast 스킨을 적용해둔터라 `skin/_contrast.scss`의 `$link-color`를 수정하니 색상 변경이 적용되었다.

### 작성날짜, 설명 한 줄에 출력
작성날짜와 설명의 폰트 크기를 키우고 한 줄에 출력하고싶다.

#### 폰트크기 조절
개발자 도구로 찍어본다. `_archive.scss`의 `.list__item .page__meta`와 `.archive__item-excerpt`에 폰트 크기를 조절한다.

#### 한 줄에 출력
루비를 아직 배우지않아서 모르겠다만 이전에 카테고리마다 글 목록 페이지를 만들때 한 카테고리에 속한 포스트들을 `archive-single.html`에 던지는(?) 구문이 있었다. (사이드바에 카테고리 생성 포스트 참고)
{%raw%}
```ruby
{% assign posts = site.categories.blog %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
```
{%endraw%}

`archive-single.html`를 살펴보면 뭔가 나올 것 같았다. html 태그들에 문자열을 넣어보면서 화면 출력을 확인한다.

**수정한 코드** - `archive-single.html`
{%raw%}
```html
{% if post.header.teaser %}
...
    </h2>
    {% include page__meta.html type=include.type %}  # 작성날짜
    {% if post.excerpt %}  # 글 설명
      <span class="archive__item-excerpt" itemprop="description">  |  {{ post.excerpt | markdownify | strip_html | truncate: 160 }}</span>
    {% endif %}
  </article>
</div>
```
{%endraw%}

* 작성날짜: `include/page__meta.html`에서 구현. 전체가 `p`태그로 감싸여있어 줄바꿈된다. `span`태그로 고쳐준다.
* 글 설명: `p`태그를 `span`태그로 고쳐주고, 작성날짜와의 간격을 위해 수직바를 추가했다.
  * 수정 전
    {%raw%}
    ```html
    <p class="archive__item-excerpt" itemprop="description">{{ post.excerpt | markdownify | strip_html | truncate: 160 }}</p>
    ```
    {%endraw%}
  * 수정 후
    {%raw%}
    ```html
    <span class="archive__item-excerpt" itemprop="description">  |  {{ post.excerpt | markdownify | strip_html | truncate: 160 }}</span>
    ```
    {%endraw%}


