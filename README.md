# Bash Bible 🐚📖
`{reference,survival}` guide to scripting in the bash shell

### Read

* [https://xero.github.io/bash-bible](https://xero.github.io/bash-bible)
* [raw github formatting](https://github.com/xero/bash-bible/blob/gh-pages/_pages/bible.md)

## History
The original [Pure Bash Bible](https://github.com/dylanaraps/pure-bash-bible) 🪦 is sadly a defunt project. When citing it to novice bash scripting colleagues, I found they often lacked the necessary fundamental context to understand it. I wished the guide itself had more beginner level information, general reference materials, & maybe more flair ;D

When the project `{author,maintainer}` [rage quit the internet](https://github.com/dylanaraps/dylanaraps/commit/811599cc564418e242f23a11082299323e7f62f8), I was worried he might delete the original repo. Instead of forking, I cloned and continued committing to the existing repo history to retain [credit to all the original authors](https://github.com/xero/bash-bible/graphs/contributors).

## Hacking
Install with [bundle](https://bundler.io), then run [jekyll](https://jekyllrb.com) build

```sh
bundle install
bundle exec jekyll build
```

### Files
* site content: [./_pages/bible.md](_pages/bible.md)
* theme css: [./_sass/bible.scss](_sass/bible.scss)
* dom skeleton: [./_layouts/bible.html](_layouts/bible.html)
* favicons & images: [./assets](assets)

## Goals
* Content ideas:
    * Script setup (shebang, chmod +x, etc)
    * Tools including [linters](https://github.com/koalaman/shellcheck), [formatters](https://github.com/mvdan/sh#shfmt), and [bash langugage server](https://github.com/bash-lsp/bash-language-server)
    * Bestow the power of [BASHOPTS](https://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html)
    * tests `[[ ]]`
    * `echo` vs `printf`
    * redirections `>, >>, <` and std`{out,in}`
    * Immutable global variables `readonly`
    * Scripting style formatting (expression breaks with `\`)
    * Closing File Descriptors
```sh
n<&- Close input file descriptor n.
0<&- or <&- Close stdin.
```
* Combine the [pure sh bible](https://github.com/dylanaraps/pure-sh-bible) into this project:
    * _the Bash Bible & Posix Psalms_

## License
The original project was MIT, and I'm sticking with that.

I've kept Dylan, added my name, and a link to the full contributors list.
