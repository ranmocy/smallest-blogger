Smallest Blogger
================

A minimal, extendable, static blogger with templates.

Powered by
[Markdown](http://daringfireball.net/projects/markdown/),
[SCSS](http://sass-lang.com/),
[Guard](https://github.com/guard/guard).

## Quick example

Example: [example/](https://github.com/ranmocy/smallest-blogger/blob/master/example/)

Usage:
```bash
git clone "https://github.com/ranmocy/smallest-blogger.git"
cd smallest-blogger/example
bundle
guard
open "http://localhost:3000"
```

## Guard version

[Guardfile](https://github.com/ranmocy/smallest-blogger/blob/master/Guardfile)
will monitor on your current folder, generate new version when you save files.
And create a server to preview on [http://localhost:3000](http://localhost:3000)

Usage:
```bash
wget "https://raw.githubusercontent.com/ranmocy/smallest-blogger/master/Gemfile"
bundle
wget "https://raw.githubusercontent.com/ranmocy/smallest-blogger/master/Guardfile"
guard
open "http://localhost:3000"
```

## Rake version

[Rakefile](https://github.com/ranmocy/smallest-blogger/blob/master/Rakefile)

Usage:
```bash
wget "https://raw.githubusercontent.com/ranmocy/smallest-blogger/master/Gemfile"
bundle
wget "https://raw.githubusercontent.com/ranmocy/smallest-blogger/master/Rakefile"
rake generate # Only generate the site
rake update   # Only publish to remote by Git
rake publish  # Both generate the site and publish to remote by Git
```

## Howto

TODO

## Philosophy

1. Hackable
2. Minimal
3. Clear

## Contribution

If you can make the base code smaller or clearer, please fork and make a pull request.

If you can extend the functionality, I will put them to `examples/` and update *Howto* section.
