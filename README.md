# Overview

[netverify.fun](http://www.netverify.fun) is a site dedicated to commentary on the state of network verification and synthesis.

# Contributing an article

Create a new markdown file with the content of your post. You can look at some examples in the `_posts` directory. The file should be named in the following format: `YYYY-MM-DD-title-of-your-post`. Then submit a new PR with the blog post you want to add.

If you want to add an image to your article, simply add the image to the `assets/images/` directory and link to it in the article markdown file.

# Building and viewing locally

You can build and view this site locally using [Jekyll](https://jekyllrb.com/).

```
gem install bundler jekyll
cd <local repository directory>
bundle install
bundle exec jekyll serve
```

Then point your browser to http://localhost:4000/
