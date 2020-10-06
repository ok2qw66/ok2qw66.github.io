---
title: First Post!
date:   2020-09-17 01:11:50 +0900
categories: 
  - How To Start
tags:
  - Blog
toc: true
toc_sticky: true
toc_label: "환경설정"
---

# 환경설정 및 테스트 서버 띄우기

github 블로그를 시작하기전 Windows 상에서 설치 및 환경 세팅을 하려고 한다.

<br>

### 1. 루비 설치하기

 [Ruby Installer Page](https://rubyinstaller.org/downloads/)에서 DevKit가 포함된 exe 파일을 다운받아 Ruby를 설치한다

(Ruby 설치 이후 DevKit 개발자 키트가 설치된다)

<br>

### 2. Start Command Prompt with Ruby 실행

Windows 탐색기에서 Start Command Prompt with Ruby를 실행한다.



![image-20200917012447987](https://user-images.githubusercontent.com/69428620/95226881-688a0a80-0838-11eb-8168-1a06fce202af.png)





------

<br>

### 3. jekyll 설치

2에서 띄운 Ruby Command 창에서 해당 명령어를 입력한다.

``jekyll new path/to/directory``

- 주의사항 : path/to/directory 경로는 새로 생성될 경로여야 한다

  (즉, 새로 생성될 폴더명을 기재한다)

```
ruby 2.7.1p83 (2020-03-31 revision a0c7c23c9c) [x64-mingw32]

C:\Users\i>cd ok2qw66  -> 내가 만들 blog 파일이 들어가는 디렉터리

# ok2qw66 폴더가 이미 존재해서 이곳에는 받을 수 없었다..
# docs 폴더를 새로 생성해 template을 받음
C:\Users\i\ok2qw66>jekyll new ./docs
Running bundle install in C:/Users/i/ok2qw66/docs...
  Bundler: Fetching gem metadata from https://rubygems.org/..........
  Bundler: Fetching gem metadata from https://rubygems.org/.
  Bundler: Resolving dependencies...
  Bundler: Using public_suffix 4.0.6
  Bundler: Using addressable 2.7.0
  Bundler: Using bundler 2.1.4
  Bundler: Using colorator 1.1.0
  Bundler: Using concurrent-ruby 1.1.7
  Bundler: Using eventmachine 1.2.7 (x64-mingw32)
  Bundler: Using http_parser.rb 0.6.0
  Bundler: Using em-websocket 0.5.1
  Bundler: Using ffi 1.13.1 (x64-mingw32)
  Bundler: Using forwardable-extended 2.6.0
  Bundler: Using i18n 1.8.5
  Bundler: Using sassc 2.4.0 (x64-mingw32)
  Bundler: Using jekyll-sass-converter 2.1.0
  Bundler: Using rb-fsevent 0.10.4
  Bundler: Using rb-inotify 0.10.1
  Bundler: Using listen 3.2.1
  Bundler: Using jekyll-watch 2.2.1
  Bundler: Using rexml 3.2.4
  Bundler: Using kramdown 2.3.0
  Bundler: Using kramdown-parser-gfm 1.1.0
  Bundler: Using liquid 4.0.3
  Bundler: Using mercenary 0.4.0
  Bundler: Using pathutil 0.16.2
  Bundler: Fetching rouge 3.23.0
  Bundler: Installing rouge 3.23.0
  Bundler: Using safe_yaml 1.0.5
  Bundler: Using unicode-display_width 1.7.0
  Bundler: Using terminal-table 1.8.0
  Bundler: Using jekyll 4.1.1
  Bundler: Using jekyll-feed 0.15.0
  Bundler: Using jekyll-seo-tag 2.6.1
  Bundler: Using minima 2.5.1
  Bundler: Using thread_safe 0.3.6
  Bundler: Using tzinfo 1.2.7
  Bundler: Using tzinfo-data 1.2020.1
  Bundler: Using wdm 0.1.1
  Bundler: Bundle complete! 6 Gemfile dependencies, 35 gems now installed.
  Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.
New jekyll site installed in C:/Users/i/ok2qw66/docs.


C:\Users\i\ok2qw66>cd docs

C:\Users\i\ok2qw66\docs>dir
 C 드라이브의 볼륨에는 이름이 없습니다.
 볼륨 일련 번호: 02B5-E00C

 C:\Users\i\ok2qw66\docs 디렉터리

2020-09-17  오전 01:12    <DIR>          .
2020-09-17  오전 01:12    <DIR>          ..
2020-09-17  오전 01:11                56 .gitignore
2020-09-17  오전 01:11               419 404.html
2020-09-17  오전 01:11               539 about.markdown
2020-09-17  오전 01:11             1,155 Gemfile
2020-09-17  오전 01:12             2,037 Gemfile.lock
2020-09-17  오전 01:11               175 index.markdown
2020-09-17  오전 01:11             2,080 _config.yml
2020-09-17  오전 01:11    <DIR>          _posts
               7개 파일               6,461 바이트
               3개 디렉터리  136,905,883,648 바이트 남음

```



### 4. _config.yml 작성

_config.yml 파일을 개인에 맞춰 설정을 변경한다.

```
title: 새요의 공부하새요 블로그
email: ok2qw66@gmail.com
description: >- # this means to ignore newlines until "baseurl:"
  Write an awesome description for your new site here. You can edit this
  line in _config.yml. It will appear in your document head meta (for
  Google search results) and in your feed.xml site description.
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
twitter_username: x
github_username:  ok2qw66

# Build settings
theme: minima
timezone: Asia/Seoul
permalink: /posts/:slug
plugins:
  - jekyll-feed
exclude:
  - README.md
  - .gitignore
  - Gemfile
  - Gemfile.lock
  - .sass-cache
  - CNAME
  - LICENSE
```



------

<br>

### 5. Front Matter 작성

블로그 글을 작성하는 규칙이다.  제목, 카테고리, 태그 등의 정보를 작성한다.

- 지금 작성한 글의 Front Matter 예시

```
title: First Post!
date:   2020-09-17 01:11:50 +0900
categories: 
  - How To Start
tags:
  - Blog
toc: true
toc_sticky: true
toc_label: "환경설정"
```

- title : 게시글 제목
- date : 작성일자
- categorites : 카테고리명 , 여러 개 작성 가능
- tags: 태그명, 여러 개 작성 가능
- ``toc: true`` : 블로그 글에서 h1~h6까지의 제목 목록 보여주기 설정
- ``toc_sticky: true`` :  toc을 사이드바에 고정(스크롤바에 상관없이 고정)
- ``toc_label`` : toc의 제목라벨

<br>

###  6. 로컬 테스트 서버 시작

``jekyll serve`` 또는 ``bundle exec jekyll serve``

```
C:\Users\i\ok2qw66\docs>jekyll serve
Configuration file: C:/Users/i/ok2qw66/docs/_config.yml
            Source: C:/Users/i/ok2qw66/docs
       Destination: C:/Users/i/ok2qw66/docs/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 1.211 seconds.
 Auto-regeneration: enabled for 'C:/Users/i/ok2qw66/docs'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.

```

<br>

### 7. 테스트 서버 접속

http://127.0.0.1:4000으로 접속하면 확인할 수 있다.

---

<br>

참고 사이트 :<br>

[https://shryu8902.github.io/_posts/2018-06-22-jekyll-on-windows/](https://shryu8902.github.io/_posts/2018-06-22-jekyll-on-windows/)<br>

[https://jekyllrb.com/docs/installation/windows/](https://jekyllrb.com/docs/installation/windows/)

[https://brianm.me/posts/how-to-make-jekyll-site-blog](https://brianm.me/posts/how-to-make-jekyll-site-blog)