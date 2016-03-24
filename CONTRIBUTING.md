# Creating a Jekyll Post

To contribute to this blog, please create a post on a topic your choice.

The file should be added to the `_posts` directory and should have file name similar to `YYYY-MM-DD-post-title.md`.
It is important that the post file name begin with the year/month/day information for the post.

The post file should include the following "frontmatter" before any content or text:
```
---
layout: post
title:  My Post
author: My Name
---
```

The `layout: post` attribute is required. 
The `title` tag is required and the value here will appear as the title of the user's browser window.
The `author` tag is optional.

The remainder of file should be the text for your blog post, written in markdown syntax.

# Style Guide

You post should start with a paragraph of lead-in text.
This is used as the excerpt on the front page of the website.

There should be no h1-style headings, please use h2 or smaller.
An h2 heading in markdown look like:
```
## My Heading
```
More leading `#` symbols mean a smaller headding.

# Submitting Your Post

Submit your post as part of a pull-request for the `gh-pages` branch of the [ROSS repository](https://github.com/carothersc/ROSS/tree/gh-pages).

# Resources

- The initial Markdown standard, from Daring Fireball: [link](http://daringfireball.net/projects/markdown/)
- GitHub Markdown guide: [link](https://help.github.com/articles/basic-writing-and-formatting-syntax/)
- Basics of Jekyll blogging: [link](http://jekyll.tips/guide/blogging/).
