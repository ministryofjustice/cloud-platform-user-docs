# Cloud platform user documentation

The documentation for users of the Ministry of Justice cloud platform. It explains how to deploy and run applications on the cloud platform.  

It's built using [Jekyll][], and hosted using [GitHub Pages][]. It
incorporates HTML, SCSS, JavaScript, and images from [GDS's Tech Docs
Template][tech-docs-template], and reworks them to work with Jekyll
instead of [Middleman][].

[gds-way]: https://github.com/alphagov/gds-way
[Jekyll]: https://jekyllrb.com
[GitHub Pages]: https://pages.github.com
[tech-docs-template]: https://github.com/alphagov/tech-docs-template
[Middleman]: https://middlemanapp.com

## Getting started

To preview the site locally, we need to use the terminal.

Install Ruby and [Bundler][bundler], preferably with a [Ruby version
manager][rvm].

[rvm]: https://www.ruby-lang.org/en/documentation/installation/#managers
[bundler]: http://bundler.io/

Once you have Ruby and Bundler set up, you can install this project's
dependencies by running the following in this directory:

```bash
bundle install
```

Docker alternative:
```bash
docker build -t jekyll -f Dockerfile-jekyll .
```

## Making changes

To make changes, edit the appropriate Markdown files in this project.
Jekyll (and therefore this site) uses [kramdown][] for its Markdown
processing.

Make sure to make changes in a branch, and issue a pull request when
you want them to be reviewed and published.

[kramdown]: https://kramdown.gettalong.org/syntax.html

## Ordering Pages

Each section that appears on the homepage (generated from `index.md`) and each page within the section has a number at the beginning of its name to indicate where in the order of pages it goes.

For instance, the "Getting started" section is named `001-getting-started.md` to indicated that it is the first section. The "Adding secrets to your app" page is named `003-add-secrets-to-your-deployment.md` showing it is the third page in its section.

When you add a new page or section:

1. Add a number to the front of the file, to show where you want it to go in the ordering
2. Make sure the rest of the files are updated to reflect that (so we don't end up with e.g. two `002`s)

## Previewing

We can preview our changes locally by running this command:

```bash
bundle exec jekyll serve --watch
```

This will create a local web server, probably at http://127.0.0.1:4000
(look for the `Server address:` line). This is only accessible on our
own computer, and won't be accessible to anyone else. It's also set up
to automatically update (thanks to `--watch`) when we make changes to
the working Markdown files.

Docker (if built above):
```bash
docker run -d --rm --name jekyll -p 8000:8000 -v $PWD:/docs jekyll
```

## Publishing changes

Because we're using GitHub Pages, any changes merged into the `master`
branch will be published automatically. Every change should be reviewed
in a pull request, no matter how minor, and we've enabled [branch
protection][] to enforce this.

[branch protection]: https://help.github.com/articles/about-protected-branches/
