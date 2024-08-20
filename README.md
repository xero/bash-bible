---
featuredLinks:
  site:
    - https://xero.github.io/bash-bible
  mirror:
    - https://lab.x-e.ro/bash-bible
---

# Bash Bible

`{reference,survival}` guide to scripting in the bash shell

## History

I do a lot of shell scripting, both personally and professionally. I've cited the, now defunct, original [Pure Bash Bible](https://github.com/dylanaraps/pure-bash-bible) a lot. But wished it had more general reference and tutorial type content. The author/maintainer [rage quit the internet](https://github.com/dylanaraps/dylanaraps/commit/811599cc564418e242f23a11082299323e7f62f8), and I was worried he might delete the original repo. So as opposed to forking, I've just continued committing on the existing repo history to [credit all the original authors](https://github.com/xero/bash-bible/graphs/contributors). I also wanted to learn more about Github Pages and jekyll theme development.

## Hacking

install with [bundle](https://bundler.io), then run [jekyll](https://jekyllrb.com) build

```sh
bundle install
bundle exec jekyll build
```

* site content: [_pages/bible.md](_pages/bible.md)
* theme css: [_sass/bible.scss](_sass/bible.scss)
* dom skeleton: [_layouts/bible.html](_layouts/bible.html)
* favicons & images: [assets](assets)

## Todo

make `_includes/toc.html` dynamically generated [like this](https://ranvir.xyz/blog/creating-table-of-content-in-jekyll-blog-without-plugin/)
