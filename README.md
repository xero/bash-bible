# Bash Bible 🐚📖
`{reference,survival}` guide to scripting in the bash shell

## History
The original, now defunct, [Pure Bash Bible](https://github.com/dylanaraps/pure-bash-bible) :headstone: was a project by a fellow nixer. When citing it to my novice-level bash scripting colleagues, they often lacked the necessary context to understand. I wished that it had more intro level, general reference materials. and maybe more flair ;D

When the project author/maintainer [rage quit the internet](https://github.com/dylanaraps/dylanaraps/commit/811599cc564418e242f23a11082299323e7f62f8), I was worried he might delete the original repo. Instead of forking, I cloned and continued committing to the existing repo history to retain some [credit to all the original authors](https://github.com/xero/bash-bible/graphs/contributors).

## Hacking

### Setup
Install with [bundle](https://bundler.io), then run [jekyll](https://jekyllrb.com) build

```sh
bundle install
bundle exec jekyll build
```

### Important Files
* site content: [_pages/bible.md](_pages/bible.md)
* theme css: [_sass/bible.scss](_sass/bible.scss)
* dom skeleton: [_layouts/bible.html](_layouts/bible.html)
* favicons & images: [assets](assets)

## Goals
* Write more entry level content
* Make [_includes/toc.html](_includes/toc.html) dynamically generated [like this](https://ranvir.xyz/blog/creating-table-of-content-in-jekyll-blog-without-plugin/)
