# Yelmo documentation website

This site is built by the Python package `mkdocs`.

To preview the site locally run `mkdocs serve`. This
will open a `localhost` that can be viewed in your browser
locally to make sure all changes make sense.

To publish the site from the master branch, simply
run `mkdocs gh-deploy`. This will build the site and
copy the files to the `gh-pages` branch online. The
new site should be available for access thereafter.

Note, to use mkdocs with the Anaconda installed version of Python
and the corresponding mkdocs module (for eg python3), use:
`python -m mkdocs gh-deploy`

Note that whenever the docs folder is updated here, it should be
manually synced with the docs folder in the Yelmo repository.
