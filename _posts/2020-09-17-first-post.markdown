---
layout: post
title:  "First Post!"
slug: first-post
date:   2020-09-17 01:11:50 +0900
description: 처음 작성하는 글
author: ok2qw66(yejin)
seo.type: BlogPosting
categories: test
---

### 첫번째 테스트 글 작성



드디어 첫번째 글을 작성할 수 있게 되었다!!

지금까지 오게 된 (첫번째 글을 작성하게 된) 과정을 작성하겠다 



참고 사이트

https://shryu8902.github.io/_posts/2018-06-22-jekyll-on-windows/

: 대부분의 글들이 linux 환경에서 설치하는 법이었다.

그래서 windows에서 git.io 만드는 법을 찾았다!

![image-20200917012447987](C:\Users\i\AppData\Roaming\Typora\typora-user-images\image-20200917012447987.png)

==> 윈도우 탐색기에서 Start Command Prompt with Ruby 실행

해당 창에서 밑에 명령어를 작성했다.



> 하라는 대로 했지만... index가 깨져 나왔다
>
> 초보인 나는 아직 무슨 문제인지 모른다... **나중에 찾아봐야됨**



------



https://brianm.me/posts/how-to-make-jekyll-site-blog

: 위에 사이트 만으로는 jekyll template이 만들어지지 않아서 찾은 방법



#### Getting Started 부분

```
jekyll new path/to/directory
```



```
ruby 2.7.1p83 (2020-03-31 revision a0c7c23c9c) [x64-mingw32]

# 내가 만들 blog 파일이 들어가는 디렉터리
C:\Users\i>cd ok2qw66

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

template을 받는데는 성공했다!



#### _config.yml 작성하기

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



> 아직 잘 모르겠으니 그냥 하라는 대로 넣어보았다.



------





#### Front Matter

```
layout: post
title: How to Make a Jekyll Site/Blog
slug: how-to-make-jekyll-site-blog
modified: 2020-05-25 01:05 CDT
description: A tutorial on making a Jekyll based site/blog.
author: brian
seo.type: BlogPosting
image: How to Make a Jekyll Site/Blog
```

> 블로그 글을 작성하는 기준? 틀?인거 같다



# 우선 시작해보자!



####  서버 시작하기

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



---

> 잘 뜬다!
>
> 문제는 이미지가 안들어간다는 점?
>
> 이건 추후에 다시 해보기로하자!