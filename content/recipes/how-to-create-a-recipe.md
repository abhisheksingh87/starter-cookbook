+++
categories = ["recipes"]
tags = ["create recipe", "new recipe"]
summary = "Steps to create a blank recipe page is outlined in this recipe"
title = "How to Create a Recipe"
date = 2020-09-01T10:55:46-05:00

+++

### Prerequisite

- `cf CLI`
- `Hugo`

### Install instructions

#### cf CLI

* Install cf CLI from https://docs.cloudfoundry.org/cf-cli/install-go-cli.html

#### Hugo

> Hugo is one of the most popular open-source static site generators, that enables user content in markdown format to be rendered in HTML and other formats.

* Install `Hugo` on your laptop from https://github.com/gohugoio/hugo/releases/download/v0.74.3/hugo_0.74.3_Windows-64bit.zip
* Update `PATH` to include the path to `hugo` executable

## Steps

1. Open a command/shell window and navigate to the `cookbook` directory

2. Create a new recipe: `hugo new recipes/a-new-recipe.md`

   * always specify `*.md` extension for the recipe file.
   * `a-new-recipe.md` is created in `content/recipes/`

1. Edit recipe using any IDE or text editor

   * avoid putting _title_ of the recipe in its body as Hugo generates one from the *title* tag.
   * to add images to a recipe, put them in `static/images` directory of the cookbook.
   * review the header in the generated recipe to adjust the tags, summary and title.

     ```toml
     categories = ["recipes"]
     tags = ["some meaningful tag1", "some meaningful tag2"]
     summary = "multi line summary should come here"
     title = "A New Recipe"
     date = 2020-09-01T10:55:46-05:00
     ```

   * be consistent with tagging. _Less is more_. Check existing tags by opening `See All Tags` link from the sidebar.

2. Preview the recipe

   * start the local Hugo server using `./localserver.bat`
   * access `http://localhost:1313` using the browser
   * the recipe should appear on the left under **Recipes**



