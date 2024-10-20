---
layout: post
title: Jekyll yat 테마 네비게이션바 순서 변경
author: 사유담
excerpt_image: ./assets/images/post_images/2024-10-20-jekyll-theme-yat-네비게이션바-순서-변경/jekyll-navigation-bar.png
categories: 잡담
tags: jekyll theme yat github-page
---

## Jekyll yat 테마 네비게이션바 순서 변경

Github 페이지는 Jekyll을 사용하여 테마를 적용할 수 있다. 여러 사이트들을 찾아다니며 마음에 드는 테마를 발견하고 적용하였는데, 그게 바로 이 페이지에 사용한 **jekyll-theme-yat**이다.
처음 테마를 올리는 것부터도 많은 시행착오가 있었다. CSS가 적용되지 않는 문제, 그리고 몇 가지 변수들이 작동하지 않는 문제들이 있었지만 주석처리를 하거나 Copilot과 ChatGPT의 도움을 받아 쉽게 풀어나갈 수 있었다.

그 시행착오 끝에 마침내 테마를 적용하고 첫 포스팅을 올렸지만, 여전히 안풀리는 문제 하나가 있었으니 바로 네비게이션바 메뉴 아이템의 순서였다.
기본적으로 이 테마는 루트 경로에 있는 html 순서에 따라 네비게이션바 메뉴를 배치한다. 그러다보니 내가 원하는 순서가 아니라 알파벳 순서대로 배치된다.

이 포스팅에서는 `jekyll-theme-yat`에서 네비게이션바 메뉴 순서를 사용자화 할 수 있도록 하는 방법을 기록할 것이다.

### 1. header.html 스크립트 살펴보기
네비게이션바는 `_includes/views/header.html` 스크립트에서 정의된다. 24번째 줄 `_includes/views/header.html`로 시작되는 블록이 바로 네비게이션바 정의 구역이다. 여기서 살펴봐야 할 부분은 17~18번째 줄과 36~41번쨰 줄까지 아래의 코드블럭들이다.

```html
<!--17~18번째 줄-->
{% raw %}
{%- assign default_paths = site.pages | where: "dir", "/" | map: "path" -%}
{%- assign page_paths = site.header_pages | default: default_paths -%}
{% endraw %}

<!--36~41번째 줄-->
{% raw %}
{%- for path in page_paths -%}
    {%- assign my_page = site.pages | where: "path", path | first -%}
    {%- if my_page.title -%}
    <a class="page-link" href="{{ my_page.url | relative_url }}">{{ my_page.title | upcase | escape }}</a>
    {%- endif -%}
{%- endfor -%}
{% endraw %}
```

`page_paths`는 18번째 줄에서 site.pages의 default_paths에 의해 루트 경로로, 그리고 site.header_pages에 의해 루트 경로에 있는 헤더 페이지들의 리스트로 구성된다.
그리고 이 페이지 리스트들은 당연히 알파벳 순서대로 정렬되어 있을 것이며, 36번째 줄의 for문에 의해 순차적으로 사용되어 네비게이션바 메뉴를 구성한다.

### 2. 스크립트 수정하기
네비게이션바 메뉴 순서를 사용자화 하기 위해 해야할 것은 다음 두 가지이다.

1. 네비게이션바 메뉴 구성 파일을 만들 것
2. 네비게이션바 메뉴 구성을 사용자가 지정한 파일에서 불러올 것

우선 네비게이션바 메뉴 구성 파일을 생성하도록 하자. 여기서는 `/_data/` 경로에 `navigation.yml` 파일로 만들었으며, 내용은 아래와 같이 작성하였다.

**/_data/navigation.yml**  

```yml
- title: Home
  path: /index.html

- title: Archives
  path: /archives.html

- title: Categories
  path: /categories.html

- title: Tags
  path: /tags.html

- title: About
  path: /about.html
```

앞으로 네비게이션바는 이 yml 파일의 목록 순서대로 만들도록 할 것이다. 이제 이 순서대로 네비게이션 메뉴를 정렬하도록 만들기 위해, `/_includes/views/header.html` 파일로 돌아와 for문을 다음과 같이 수정한다.

**/_includes/views/header.html**  

```html
{% raw %}
{% for page in site.data.navigation %}
    <a class="page-link" href="{{ page.path | relative_url }}">{{ page.title | upcase | escape }}</a>
{% endfor %}
{% endraw %}
```

여기서 site.data.navigation은 `/_data/navigation.yml` 파일을 의미한다. 교체된 for문은 navigation.yml 내부에 있는 리스트들을 하나씩 가져와 page 변수로 정의한다. yml의 각각의 리스트 아이템에는 title과 path 요소가 있다. 그러므로 page.path를 통해 네비게이션 버튼의 링크를 연결하고, 네비게이션 버튼 글자는 page.title을 모두 대문자로 표기(upcase)하여 표시한다.

그리고 17~18번째 줄의 코드는 더 이상 필요하지 않으므로 삭제하였다.

**/_includes/views/header.html**  

```html
{% raw %}
<!--17~18번째 줄 삭제
{%- assign default_paths = site.pages | where: "dir", "/" | map: "path" -%}
{%- assign page_paths = site.header_pages | default: default_paths -%}
-->
{% endraw %}
```

여기에 나는 아래의 코드를 `nav` 블록 앞에 추가하여 navigation.yml이 있는지 여부를 사전에 확인하도록 하였다.

**/_includes/views/header.html**  

```html
{% raw %}
{%- if site.data.navigation -%}
    <nav class="site-nav">
{%- endif -%}
{% endraw %}
```

최종적으로 nav 블록의 전체 모습은 아래와 같이 변하였다.

**/_includes/views/header.html**  

```html
{% raw %}
{%- if site.data.navigation -%}
    <nav class="site-nav">
        <input type="checkbox" id="nav-trigger" class="nav-trigger" />
        <label for="nav-trigger">
            <span class="menu-icon">
                <svg viewBox="0 0 18 15" width="18px" height="15px">
                    <path d="M18,1.484c0,0.82-0.665,1.484-1.484,1.484H1.484C0.665,2.969,0,2.304,0,1.484l0,0C0,0.665,0.665,0,1.484,0 h15.032C17.335,0,18,0.665,18,1.484L18,1.484z M18,7.516C18,8.335,17.335,9,16.516,9H1.484C0.665,9,0,8.335,0,7.516l0,0 c0-0.82,0.665-1.484,1.484-1.484h15.032C17.335,6.031,18,6.696,18,7.516L18,7.516z M18,13.516C18,14.335,17.335,15,16.516,15H1.484 C0.665,15,0,14.335,0,13.516l0,0c0-0.82,0.665-1.483,1.484-1.483h15.032C17.335,12.031,18,12.695,18,13.516L18,13.516z"/>
                </svg>
            </span>
        </label>

        <div class="trigger">
        {%- for page in site.data.navigation -%}
            <a class="page-link" href="{{ page.path | relative_url }}">{{ page.title | upcase | escape }}</a>
        {%- endfor -%}

        {%- assign name = 'translate_langs' -%}
        {%- include functions.html func='get_value' -%}
        {%- assign translate_langs = return -%}
        {%- if translate_langs.size > 0 -%}
            {%- assign name = 'lang' -%}
            {%- include functions.html func='get_value' default='en' -%}
            {%- assign lang = return -%}
            <div class="page-link" style="display: inline;">
                {%- include extensions/google-translate.html -%}
            </div>
        {%- endif -%}
        </div>
    </nav>
{%- endif -%}
{% endraw %}
```

이제 수정사항을 커밋하고 푸시하고 기다리면 깃허브 페이지의 네비게이션바가 원하는대로 정렬된 것을 볼 수 있다.
