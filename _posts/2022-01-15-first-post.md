---
title: "포스팅 연습 & 블로그 고치기"
# folder: "blog"
excerpt: "블로그 고치기"

categories:
  - blog
tags:
  - [blog]

# author_profile: true
# sidebar:
#   nav: "sidebar_category"

toc: true
toc_sticky: true

date: 2022-01-15
last_modified_at: 2021-01-15
---

# 포스트 작성하기

블로그 생성 참고 문서: [minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/) - fork 방식 이용


## h2
### h3
#### h4


* TODO
  * [x] 폰트 사이즈 정리:
    * ~/_sass/minimal-mistakes/_reset.scss의 21 line
    * ```scss
      @include breakpoint($x-large) {
        font-size: 15px;
      }
      ```
  * [x] 블로그 여기저기가 폰트 사이즈에 종속되어있다.
    * [x] 블로그 우상단의 이미지와 블로그 타이틀 크기 조절
      * ~/_sass/minimal-mistakes/_masthead.scss
      * .site-title: 타이틀 - font-size 추가
      * .site-log: 로고 이미지 - max-height 조절
      * .site-subtitle: 서브타이틀 - font-size 조절 (type-size-8: 0.625em)
  * [x] $(3x + 5)$ mathjax 적용하기
    * 출처: [수염이 멋진 엔지니어 블로그](https://www.janmeppe.com/blog/How-to-add-mathjax-to-minimal-mistakes/)
    * _include/scripts.html에 아래 코드 추가하기
    * ```html
      <script type="text/javascript" async
        src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML">
      </script>

      <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
          extensions: ["tex2jax.js"],
          jax: ["input/TeX", "output/HTML-CSS"],
          tex2jax: {
            inlineMath: [ ['$','$'], ["\\(","\\)"] ],
            displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
            processEscapes: true
          },
          "HTML-CSS": { availableFonts: ["TeX"] }
        });
      </script>
      ```
  * [ ] 사이드바 추가하기
    * [ ] 카테고리 추가하기
    * [ ] 태그 추가하기
  * [x] 포스트 날짜 형식 수정
    * 출처: [민한님 블로그](https://minhaaan.github.io/blog/change_dateFormat/)
    * `__include/page__date.html`, `__include/page__meta.html`의 날짜 포멧 수정

* 기타
  * 페이지 하단의 태그, 카테고리 크기: _sass/minimal-mistakes/_page.scss
    * ```scss
      .page {
        ...
        .page__inner-wrap {
          ...
          
          .page__taxonomy {  //추가하기
            font-size: 1.25em;
          }
        }
      }
      ```

