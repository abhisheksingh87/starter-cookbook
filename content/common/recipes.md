+++
draft = false
categories = ["recipes"]
description = "How to Add a New Recipe to the Cookbook"
date = "2021-01-29T14:45:27-06:00"
tags = ["Hugo", "Markdown", "meta"]
title = "Adding and Updating Recipes"
weight = 10
+++

## First Things First

You should be familiar with [Markdown](https://daringfireball.net/projects/markdown/syntax) and [Git](https://git-scm.com/).

Hugo uses standard Markdown with the extensions provided by the [markdown library](https://github.com/russross/blackfriday#extensions).  There are some slight syntax differences compared to GitHub Flavored Markdown (GFM).

You may want to use a programming editor that uses monospaced text and supports Unix line endings, vi on a Linux host, or even your IDE may work.

## Contribution Guidelines

### Required

- Context (who would use this and why)
- Explanation or link to procurement/onboarding process for any products used (ex - [TSC]())

### Not Allowed

- [Abandonware](https://en.wikipedia.org/wiki/Abandonware)
- Links to personal repos. Any guides supplemented with example projects must use innersourced application repos (i.e.: github.wellsfargo.com/appxxxx/some-project).
- Usage of tools that are limited/restricted for whatever reason (licensing, security, etc.)
- Processes that are non-compliant or lead to non-compliance (e.g., modifying open source, storing passwords in plain text)

### Conventions

**File names:** all lower-case with hyphens in place of spaces

**Tags:** follow conventional/industry spelling, grammar, and capitalization when applicable to avoid duplication (e.g., SQL, PaaS, public cloud). Tags will be reviewed as part of the merge request process. 
Check the [Index By Subject](/collections/index-by-subject) for some commonly-used ones.

**Markdown:** mentioned above, this is a different variation of markdown than GFM. While HTML syntax is supported for additional formatting options, Markdown is preferred for maintainability and readability purposes as well as visual consistency across the cookbook.  
Tools like [pandoc](https://pandoc.org/) will convert from other formats to a Markdown file but still embed HTML.

### CodeOwners

You can check out the [CODEOWNERS file](https://github.wellsfargo.com/app-ebst/wf-starter-cookbook/blob/master/CODEOWNERS) for contact info.  

### Should I create a separate cookbook?

Probably not.  Maybe you are saying to yourself: *But the cookbook doesn't have what I need.*

That's an opportunity for improving the cookbook.  
If the information isn't there today or can be improved, you can help us add it so that others can benefit tomorrow. Please get in touch with _EA_ Team

### Got it.  How can I contribute to the cookbook?

Continue reading this recipe.  

### Create Your Copy of the Project

You do not need to be a member of this project to make contributions.

*NOTE:* This will walk you through setting up a triangular workflow.  It is one of a few common version control workflows.  This one is well-suited for projects like the cookbook that are _innersourced_ and require changes through merge requests from other repos.  A different workflow may be more appropriate for projects of which you are a member.

The triangular workflow uses two separate copies of the repository that you control: one on your workstation and one in GitHub.

![Triangular Workflow](/images/recipes/triangular-workflow.png)
[Image source.](https://github.blog/2015-07-29-git-2-5-including-multiple-worktrees-and-triangular-workflows/#improved-support-for-triangular-workflows)

Clone the repo to your workstation:

```bash
$ git clone git@github.wellsfargo.com/app-ebst/wf-starter-cookbook.git

# or...

$ git clone https://github.wellsfargo.com/app-ebst/wf-starter-cookbook
```

Go to [the repo](https://github.wellsfargo.com/app-ebst/wf-starter-cookbook) and click on the Fork button to create a copy in your personal project area.

Modify the push location for your remote (`origin`):

```bash
$ git remote set-url --push origin git@github.wellsfargo.com/MY_EMP_ID/wf-starter-cookbook.git

# or...

$ git remote set-url --push origin https://github.wellsfargo.com/MY_EMP_ID/wf-starter-cookbook.git
```

...replacing `MY_EMP_ID` with your employee ID.

Your remote configuration should now look something like this:

```bash
$ git remote -v
origin	https://github.wellsfargo.com/app-ebst/wf-starter-cookbook.git (fetch)
origin	https://github.wellsfargo.com/app-ebst/wf-starter-cookbook.git (push)
```

This configuration will pull upstream changes from the source cookbook and push all changes (upstream plus any changes that you make) to your personal copy in GitHub.

## Running Hugo

Download and install [Hugo](https://gohugo.io/getting-started/installing/).

Then to create a new recipe you can (using your own title as my-title).

```bash
$ hugo new recipes/my-title.md

# or if you're a vi user

$ vi $(hugo new recipes/my-title.md)
```

`hugo` will print the absolute path to the file it generated.  You can edit this
template to create your recipe.  It will look something like:

```markdown
+++
draft = false
title = "My Title"
description = "Recipe Summary"
categories = ["recipes"]
tags = ["foo"]
date = 2018-04-03T10:40:56-05:00
+++

brief intro

## Section 1
Explain to others how to accomplish a general task and try to include
as much code and as many commands and other examples as possible.

## Section 2
Here's some more info

### Subsection
```

Make sure you update the description and add any categories or tags
that are appropriate.  Below the second set of `+++` you can markdown away to
share your experience with the rest of the development community
here at WellsFargo.


When you are done you can check for errors and preview your work on a local server:

```bash
$ hugo serve
```
For advanced usage, you can check out the Hugo documentation [here](https://gohugo.io/getting-started/).

## Prepare Your Changes for Submission

Make sure that your changes conform to the above guidelines. When you're happy, commit your changes.

```bash
$ git add content/recipes
$ git commit -m 'Adding recipe for My Title'
```

And then push to your fork:

```bash
$ git push
```

## Submit Your Merge Request

Satisfied with a job well done, you can put in a [merge request](https://github.wellsfargo.com/help/user/project/merge_requests/index.md)
from your branch of your fork to `master` of the [main
repo](https://github.wellsfargo.com/app-ebst/wf-starter-cookbook).

The merge request will be reviewed and either approved or commented on soon.  Changes and adjustments might be necessary and any additional commits/merges into the source branch (your branch) will be reflected in the open merge request.

## Regular Maintenance for your repositories

It's common to initially fork a project when you want to make your first set of changes and then forget it's there until there are some other changes that you want to make.  When making changes at a later time, simply pull upstream updates to mitigate merge conflicts when starting new work:

```bash
$ git pull origin master

# make the best recipe ever

$ git push origin master
```
