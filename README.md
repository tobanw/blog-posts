# Blog posts repo for jekyll site

Rather than having one repo for a `jekyll` site and mixing up the `git` history with commits for the site and the content, I'm using a `git` submodule for posts.
See [this guide](http://jry.io/posts/make-your-jekyll-blog-awesome-with-git-submodules/) for an overview of the setup.

## Usage

To keep things simple, write posts in the actual repo.
Don't edit the submodule.
The usual workflow is:

1. In the `blog-posts` repo: write posts and push to the remote when ready
2. In the main site repo: grab updated submodule with `git submodule update --remote`, and then commit and push the change to publish to your site
