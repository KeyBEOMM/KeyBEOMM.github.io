---
title: "GitHub Pages 시작하기"
date: 2025-05-06 18:08:00 +0900
categories: [Hit the SW, GitHub pages]
tags: [gitHub pages, jekyll, chirpy, velog]
description: Creataing a GitHub pages site & Customizing it! 
media_subpath: /assets/post/HelloBlog/
---

![GitHub Page 사진](githubpage_white.png){: w="700" h="700" }{: .light }
![GitHub Page 사진](githubpage_dark.png){: w="700" h="700" }{: .dark }


## Markdowm Syntax

> 프롬프트 박스: 정보 박스
{: .prompt-info }


> 프롬프트 박스: 팁 박스
{: .prompt-tip }


> 프롬프트 박스: 경고 박스
{: .prompt-warning }


> 프롬프트 박스: 위험 박스
{: .prompt-danger }



### 코드 박스
Markdown symbols * '```'* can easily create a code block as follows:
```cpp
cout << HelloWorld! << endl;
```



## 문제 상황!!

GitHub 블로그 테마인 Chirpy Starter를 사용했다. Chrispy에는 Velog 소셜 버튼이 기본 제공되지 않는다.

먼저 소셜 아이콘 렌더링하는 파일 찾을 찾자. `_includes/` 폴더가 없다! 보통 테마들은 `_includes/` 안에 `social.html`, `footer.html`, `header.html` 등의 코드 파일들이 존재하고 이를 수정해서 커스텀 가능하다.

Chripy Strarter는 이건 Chirpy 테마를 RubyGems로 설치한 버전이다

그래서 프로젝트 폴더에는 `_data`, `_layouts`, `_includes`, `_sass`, `assets` 정도만 있고, 나머지 "진짜 Chirpy 테마의 소스코드" 는 Gem 패키지 안에 숨겨져 있다.



### 해결 방안
Chirpy Starter가 아닌 Chirpy 테마에서 직접 복사

순서
1. 현재 설치된 chirpy 테마 폴더 경로 찾기
```
bundle info --path jekyll-theme-chirpy
```

2. 그 경로에 가서, 필요한 폴더를 복사하기

    > 복사 대상 
    >   * `_includes/`
    >   * `_ layouts/`

3. 소셜 아이콘 렌더링하는 파일 찾을 찾기

본인의 경우 `_includes/sidebar.html`

4. 코드 수정

{% raw %}
```html
{% for entry in site.data.contact %}
  {% case entry.type %}
    {% when 'github', 'twitter' %}
      ...
    {% when 'email' %}
      ...
    {% when 'rss' %}
      ...
    {% else %}
      {% assign url = entry.url %}
  {% endcase %}
{% endfor %}
```
{% endraw %}

이 부분에서 Velog를 따로 `when`으로 추가하면 된다.


수정 코드

{% raw %}
```html
{% for entry in site.data.contact %}
  {% case entry.type %}
    {% when 'github', 'twitter' %}
      {%- capture url -%}
        https://{{ entry.type }}.com/{{ site[entry.type].username }}
      {%- endcapture -%}
    {% when 'velog' %}
      {%- capture url -%}
        https://velog.io/@{{ site.velog.username }}
      {%- endcapture -%}
    {% when 'email' %}
      {% assign email = site.social.email | split: '@' %}
      {%- capture url -%}
        javascript:location.href = 'mailto:' + ['{{ email[0] }}','{{ email[1] }}'].join('@')
      {%- endcapture -%}
    {% when 'rss' %}
      {% assign url = '/feed.xml' | relative_url %}
    {% else %}
      {% assign url = entry.url %}
  {% endcase %}
{% endfor %}
```
{% endraw %}

> velog 링크 형식은 https://velog.io/@username 다른 링크들과 달리 `@`가 들어간다
{: .prompt-info }


그리고 icon, 소셜 타입을 관리하는 `_data/contact.yml`과 username과 ,link 관리하는 `_config.yml`도 수정해주자.

**'_data/contact.yml'**

icon은 'fa-solid fa-pen-to-square' 사용했다.
```yaml
- type: velog
  icon: "fa-solid fa-pen-to-square"
```
**'_config.yml'**

username 추가
```yaml
velog:
    username: kgbkelvin # change to your Velog username
```

link 추가
```yaml
links:
    - https://velog.io/@username 
```


5. 저장 후 로컬에서 확인해보기!
* 루비(Ruby) 설치
설치 확인

```
ruby -v
```

* 필수 패키지 설치(bundler, jekyll)
설치 및 확인

```
gem install bundler
gem install jekyll

jekyll -v
```

* Bundle 설치

```
bundle install
```


* 서버 실행

```
bundle exec jekyll serve
```


6. 잘 되면 add / commot / push

