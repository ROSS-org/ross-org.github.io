This is the website branch for ROSS.
The website is created using [Jekyll](http://jekyllrb.com) and [GitHub Pages](https://pages.github.com).

## Contributing to the Blog

See the guidelines in [CONTRIBUTING.md](https://github.com/ross-org/ross-org.github.io/blob/master/CONTRIBUTING.md).


## Local Website Development

1. Make sure your machine is properly configured with the Jekyll and GitHub Pages ruby gems by following [these instructions](https://help.github.com/articles/setting-up-your-pages-site-locally-with-jekyll/).
2. Start the local webserver using the command `bundle exec jekyll serve -w --baseurl=`
3. Visit the site at [localhost:4000](http://localhost:4000).

More information can be found [here](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/).

## A Note on Doxygen

The doxygen for ROSS is automatically generated and updated through git-hooks.
These files are stored in the `ROSS-docs/docs/html` directory.
To change the Doxygen settings see the [Doxyfile.user.in file](https://github.com/ross-org/ROSS/blob/master/docs/Doxyfile.user.in) in the ROSS/master branch.
