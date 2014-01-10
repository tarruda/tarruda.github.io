---
title: Giving back
author: tarruda
date: 2014-01-09
template: article.jade
---

For more than 10 years I've been improving my IT skills - most of which are
related to computer programming - and it's safe to say that at least 90% of my
knowledge came from internet sources. <span class="more">

As a natural consequence of learning, I've developed a few ideas of my own,
ideas that may be interesting or useful to others, but lacked a simple way to
share these ideas, which was the main motivation for me to start this blog.

Another reason is to receive feedback from readers who share my interests, which
will help me to hone my skills and perhaps open my mind create more cool things.
Finally, it will improve by english writing, which is kinda poor right now.

[Wintersmith](https://github.com/jnordberg/wintersmith) enabled me to get
started very quickly, these were the basic steps:

```sh
# install wintersmith and scaffold the blog
$ npm install -g wintersmith
$ wintersmith new ~/my-blog
$ cd ~/my-blog
# move everything to a subdirectory and set the "output" option of config.json
# to '../'(the generated pages need to be in the repository root for github)
$ mkdir src
$ mv config.json contents node_modules package.json plugins readme.md templates src
$ (cd src && wintersmith preview) # preview the blog
$ (cd src && wintersmith build)   # generate the blog
$ git init
$ git remote add origin git@github.com:tarruda/tarruda.github.io
$ git add .
$ git commit
$ git push --set-upstream origin master
```

Thats it for now. If you are a computer geek who enjoys reading techinal
articles, subscribe or check back every once in a while and I will be sure
to post interesting things.
