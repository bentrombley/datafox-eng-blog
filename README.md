This is the DataFox engineering blog, hosted on github pages.

To create and edit blog posts:

## Initial Setup

- git clone this repo locally
- install jekyll

### Running locally:

    cd ~/datafoxco.github.io
    jekyll serve --drafts

Visit [localhost:4000](http://localhost:4000) in your browser.

## Writing Posts

Create a new `.md` file in the `_drafts` folder and edit in markdown.

At the top add a section

    ---
    layout: post
    title:  "My Blog Post Title"
    date:   2017-01-08
    categories: css, clean code
    ---

Add any images to the `img` directory and reference like this in your post:

    <img src="/img/path/to/my/file.jpg" width="100%" />

You can preview your post by visiting [localhost:4000](http://localhost:4000) in your browser.


## Submitting for Review

Commit your new file in the `_drafts` folder and push to the repo or Phabricator for others to see.


## Publishing

Use `git mv` to move your draft file to the `_posts` folder and put the publish date at the beginning like `2017-01-01-my-post.md`.

Commit your change and push to the repo, and it will be automatically built and live within 5 minutes (usually faster).


