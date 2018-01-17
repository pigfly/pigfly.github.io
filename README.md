My Blog
=======

My [blog](https://pigfly.github.io/).

[![Travis](https://img.shields.io/travis/rust-lang/rust.svg)](https://travis-ci.org/pigfly/pigfly.github.io.svg?branch=master)
[![Ruby](https://img.shields.io/badge/ruby-2.4.0-orange.svg)](https://www.ruby-lang.org/en/downloads/)
[![Website](https://img.shields.io/website-up-down-green-red/http/shields.io.svg?label=pigfly.github.io)](https://pigfly.github.io)
[![Plugin on redmine.org](https://img.shields.io/redmine/plugin/stars/redmine_xlsx_format_issue_exporter.svg)](https://pigfly.github.io)


Setup
-----

Note this project is built on top of [Jekyll](https://jekyllrb.com/), please see [here](http://jekyllrb.com/docs/installation/) for how to setup jekyll installation.

```shell
git clone https://github.com/pigfly/pigfly.github.io.git
cd pigfly.github.io
gem install jekyll
gem install bundler
bundle install
jekyll serve --watch
```

Then navigate to [http://127.0.0.1:4000/](http://127.0.0.1:4000/).

A Grunt environment is also included. There are a number of tasks it performs like minification of the JavaScript, compiling of the LESS files, Run the grunt default task by entering `grunt` into your command line which will build the files. You can use `grunt watch` if you are working on the JavaScript or the LESS.

You can run `jekyll serve --watch` and `grunt watch` at the same time to watch for changes and then build them all at once.

License
-------

[![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png)](http://creativecommons.org/licenses/by/4.0/)

This work by [Alex Jiang](https://pigfly.github.io) is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

Code I've written is [licensed](/LICENSE) under MIT. Other components such as [Bootstrap](http://getbootstrap.com) have their own licenses.

Thanks
------

Special thanks to [iT Man](https://itman.design), who continue to provide high quality computing environment for free.

Thanks to the following people and projects:

- [loadCSS](https://github.com/filamentgroup/loadCSS)
- [Font Awesome](http://fortawesome.github.io/Font-Awesome/icons/)
- [This colour](http://www.colourlovers.com/color/398CCC/Walton)
- [This colour scheme](http://www.colourlovers.com/palette/869489/Caribbean_Dusk)
- [vertical timeline jquery thingy](http://www.jqueryscript.net/other/Responsive-Vertical-Timeline-With-jQuery-CSS3.html)
- [@CloudyConway](http://twitter.com/CloudyConway)
- [IcoMoon](https://icomoon.io)
- [CloudFlare](http://cloudflare.com)

Server Setup
------------

The site is served through CloudFlare's CDN. The CDN caches everything on edges. These edges respect the caching header set on individual files. CloudFlare also sets a browser cache expiration of 30 minutes for all content (if a longer one is not specified, see below).
