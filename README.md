## Indigo Minimalist Jekyll Template - [Demo](http://sergiokopplin.github.io/indigo/)
[![Build Status](https://travis-ci.org/sergiokopplin/indigo.svg?branch=gh-pages)](https://travis-ci.org/sergiokopplin/indigo)

This is a simple and minimalist template for Jekyll for those who likes to eat noodles.

***

<p align="center">
    <b><a href="README.md#what-has-inside">What has inside?</a></b>
    |
    <b><a href="README.md#setup">Setup?</a></b>
    |
    <b><a href="README.md#settings">Settings</a></b>
    |
    <b><a href="README.md#tests">Tests</a></b>
    |
    <b><a href="README.md#showcase">Showcase</a></b>
    |
    <b><a href="README.md#contributions">Contributions</a></b>
    |
    <b><a href="README.md#problems">Problems</a></b>
    |
    <b><a href="README.md#update-your-fork">Update your fork</a></b>
    |
    <b><a href="README.md#license">License</a></b>
</p>

![Screenshot](https://raw.githubusercontent.com/sergiokopplin/indigo/gh-pages/screen-shot.png)

---

## What has inside?

- [Gulp](http://gulpjs.com/) && [BrowserSync](https://www.browsersync.io/)
- [Stylus](http://stylus-lang.com/) with [RSCSS](http://rscss.io/) Methodology
- [SVG](https://www.w3.org/Graphics/SVG/)
- Tests with [Travis](https://travis-ci.org/)
- No JS
- Google Speed: [98/100](https://developers.google.com/speed/pagespeed/insights/?url=http%3A%2F%2Fsergiokopplin.github.io%2Findigo%2F)

# Setup

0. :star: to the project. :metal:
1. [Install Jekyll](http://jekyllrb.com)
2. [Install NodeJS](https://nodejs.org/)
3. [Install Bundler](http://bundler.io/)
4. Fork the project [Indigo](https://github.com/sergiokopplin/indigo/fork)
5. Edit `_config.yml` with your data.
6. `bundle install`
7. `npm install`
8. `npm install -g gulp`
9. `gulp`
10. open in your browser: `http://localhost:3000`

> [Arti Annaswamy](https://github.com/aannasw) wrote a really nice tutorial of "how to install jekyll". Here's the [part 1](http://artiannaswamy.com/build-a-github-blog-part-1) and [part 2](http://artiannaswamy.com/build-a-github-blog-part-2). :metal:

# Settings

You must fill some informations on `_config.yml` to customize your site.

```
name: John Doe
bio: 'A Man who travels the world eating noodles'
picture: 'assets/images/profile.jpg'

url: http://YOURUSER.github.io
# like: http://sergiokopplin.github.io
permalink: /:title/

analytics: 'UA-MYANALYTICS'
disqus: mydisqus
facebook: myfacebook
twitter: mytwitter

...

and lot of other options, like width, projects, pages, read-time, tags, relateds, animations, etc.
```

## Tests

You can test your app with:

```bash
bundle exec htmlproof ./_site --verbose
````

If you want travis tests, you can take a look at:
- [http://www.raywenderlich.com/109418/travis-ci-tutorial](http://www.raywenderlich.com/109418/travis-ci-tutorial)

## Showcase

- [fohlen.github.io](http://fohlen.github.io/) by @[Lennard Berger](https://github.com/Fohlen)
- [gedankenstuecke.github.io](http://gedankenstuecke.github.io) by @[Bastian Greshake](https://github.com/gedankenstuecke)
- [artiannaswamy.com](http://artiannaswamy.com/) by @[Arti Annaswamy](https://github.com/aannasw)

> open a [pull-request](https://github.com/sergiokopplin/indigo/pulls) if you want your site in this awesome list

## Problems?

Tell me on github or open a [issue](https://github.com/sergiokopplin/indigo/issues/new).

### Update your fork.

- [github.com/articles/syncing-a-fork/](https://help.github.com/articles/syncing-a-fork/)

---

## License

[MIT](http://kopplin.mit-license.org/) License © Sérgio Kopplin
