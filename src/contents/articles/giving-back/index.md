---
title: Giving back
author: tarruda
date: 2014-01-09
template: article.jade
---


For more than 10 years I've been improving my IT skills - most of which are
related to computer programming - and it's safe to say that at least 90% of my
knowledge came from internet sources.

As a natural consequence of learning, I've developed a few ideas of my own,
ideas that may be interesting or useful to others, but I lacked a simple way to
share these ideas. This is one of the reasons that motivated me to start this
blog. Another reason is to receive feedback from others who share my interests,
which will help me evolve in my art and perhaps let create more cool things.

[Wintersmith](https://github.com/jnordberg/wintersmith) enabled me to get
started very quickly, these were the basic steps:

```sh
$ npm install -g wintersmith
$ wintersmith new ~/my-blog
$ cd ~/my-blog
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

Thats it for now. If you are a computer geek, be sure to subscribe or check
back every once in a while to read some interesting technical articles.
