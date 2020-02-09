---
layout: post
title: 'Make this website'
summary: 'Learn how this website was made.'
category: 'walkthrough'
---

# What Makes a Website

HTML + CSS

# Why Use a Static Site Generator?

Remove a lot of the tedium of doing the same thing over an over by hand.
Use a lot of starter code that someone else has written for us so we can focus on the website content right away and slowly adjust the 
styles and whatnot over time.

# What Static Site Generator are We Using?

[Jekyll](https://jekyllrb.com/)

This site is going to talk a lot about the [Ruby](https://www.ruby-lang.org/en/) programming language
so it is only fitting that our static site generator is written in Ruby.

# Initial Setup

1. Follow the installation instructions on the Jekyll site.
1. Initialize [git](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control) repository
```bash
git init
git add -A
git commit -am 'Start new jekyll site'
```
1. Update the _config.yml file with info about the site.
    + title, email, description, url, github_username
    + remove twitter_username
    + remove [minima theme](https://github.com/jekyll/minima)
    + add [jekyll-seo-tag](https://github.com/jekyll/jekyll-seo-tag) to plugins

1. Update the `Gemfile` to include `jekyll-seo-tag`
```ruby
group :jekyll_plugins do
  ...
  gem 'jekyll-seo-tag', '2.6.1'
end
```

1. Copy all of the assets from the starter theme plugin (minima) into our project.
See [Overriding theme defaults](https://jekyllrb.com/docs/themes/#overriding-theme-defaults)
In the terminal:
```bash
cp -r $(bundle show minima)/_layouts
cp -r $(bundle show minima)/_includes
cp -r $(bundle show minima)/_sass
cp -r $(bundle show minima)/assets
```
1. Start updating theme defaults
    1. I'm not going to use Google Analytics
        + remove lines 8-10 in `_includes/head.html`
        + `rm _includes/google-analytics.html`
    1. I'm not going to use Disqus Comments
        + Remove lines 33-35 in `_layouts/post.html`
        + `rm _includes/disqus_comments.html`
    1. The only Social features I am going to use are Github and Youtube
        + `rm _includes/icon-twitter.html _includes/icon-twitter.svg`
        + Remove all of the lines that aren't about Github, Youtue, or RSS in `_includes/social.html`
        + Ditto for all svg icons in `assets/minima-social-icons.svg`
1. Make sure none of our changes broke anything by running `bundle exec jekyll serve` and visiting the site on `localhost:4000`
1. Commit changes to git
```bash
git add -A
git commit -am 'Copy and customize Minima theme'
```

# Changing Syntax Highlighting

Jekyll uses [Rouge](https://github.com/rouge-ruby/rouge) to do syntax highlight on blocks of code in Markdown.
I am going to use the Thankful Eyes theme.

In terminal:
```bash
gem install rouge
```

In ruby console (irb)
```ruby
File.open('_scss/minima/_syntax-highlighting.scss', 'wb') do |f| 
  f << Rouge::Themes::ThankfulEyes.render(scope: '.highlight') 
end
```

I like this for blocks of code but it's too strong to apply the exact same styles to inline blocks.
Instead I am going to translate the background color to rgba and set the alpha value to something less than 1 so that there's enough
of a difference to catch the eye but not too much to make it a jarring transition from the white-backgrounded text that will be right next to it.

In `_scss/minima/_base.scss`
```scss
pre, 
code {
  &.language-plaintext {
    background-color: rgba(22, 52, 71, 0.03);
  }
}
```

# Changing Fonts

I like sans-serif fonts for headers but I prefer serif for body copy.

1. In `_sass/minima.scss` 
    + delete `$base-font-family`
    + add `$sans-serif-font-family: sans-serif;`
    + add `$serif-font-family: serif;`
1. In `_sass/minima/_base.scss` 
    + change `$base-font-family` to `$serif-font-family` in the rules for body
    + add `font-family: $sans-serif-font-family` to the rules for headings and anchors

# Bash command to start posts

Writing posts with the correct file name and YAML frontmatter is a bit tedious so I created a shell script to do that for me.

I use [zsh](http://zsh.sourceforge.net/) and [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) so to add this function I...

1. Navigate to `~/.oh-my-zsh/custom`
1. Create a new file called `jekyll-blog.zsh` and open it in my editor
1. Write a function called `new_post` that will take three arugments
    1. The directory where the post should go
    2. The title of the post
    3. A short summary of what the post is about

This function should...
    1. use the title to generate a file name that will be the slug like `2020-02-08-my-title.md`
    1. add the title and the summary to the YAML frontmatter of the file
    1. write the file in the directoy specified
    1. open my text editor ([Emacs](https://www.gnu.org/software/emacs/)) in the directory where I created the file
  
```bash
function new_post() {
  cd ~/$1
  SLUGIFIED="$(echo -n "$2" | sed -e 's/[^[:alnum:]]/-/g' | tr -s '-' | tr A-Z a-z)"
  SLUG=$(date +"%Y-%m-%d"-$SLUGIFIED.md)
  cat <<front_matter > $SLUG
---
layout: post
title: '$2'
summary: '$3'
---

### Words Go Here
front_matter

  emacsclient -t .
}
```

## Learn about the Bash commands used in this function:

+ `man echo`
+ `man sed`
+ `man tr`
+ `man date`
+ `man cat`

`man` is the command to pull up the manual pages for another commands, but you can also use it to learn about itself.

+ `man man`

## Using the function

```bash
new_post ~/NonBreakingSpace/blog/_posts/ "Make this Website" "Learn how this website was made."
```

# Writing the Post

Post contents are written in [Markdown](https://www.markdownguide.org/getting-started/)

With `bundle exec jekyll serve` running while I write the post I can see the page refresh on my site and keep an eye on the results to make sure I am formatting my markdown correctly.

Once I am done writing I commit the post to git.

```bash
git commit -A
git commit -am 'First draft of first post'
```

# Publishing the Website

Using [Github Pages](https://pages.github.com/)

Project site
Start from scratch

Create a repository
Copy the git URL
Set the repo as a remote
```bash
git remote add origin git@github.com:pathouse/non-breaking-space.git
```

In `Gemfile`
1. Remove lines for `gem "jekyll"` and `gem "minima"`
1. Inside `group :jekyll_plugins` add `gem "github-pages"`

Then run `bundle update`

Build the site into the output folder used by Github Pages

```bash
jekyll build -d docs
```

Commit all changes

```bash
git add -A
git commit -am 'Deploying with Github Pages'
```

Push your changes to github

```bash
git push origin master
```
In in the Github Repo go to settings, scroll down to Github pages - select `master branch /docs folder` as the source for your site