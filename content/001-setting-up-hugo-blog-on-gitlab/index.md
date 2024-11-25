+++
Tags = ["General"]
Description = ""
date = 2017-03-03
title = "Setting up a Hugo-based Blog on GitLab"
menu = ""
[taxonomies]
tags = ["General"]
+++

This was my first time setting up a blog on GitLab and it was a very easy process.  I'm using [ Hugo](https://gohugo.io/) and there are some straight-forward instructions for creating a bare-bones site and deploying it to a project: [Hosting Hugo on GitLab](https://gohugo.io/tutorials/hosting-on-gitlab/)

I did run into one problem though that I wanted to share with the inter-tubes.  When I first pushed my one-post-blog project to test, I wasn't able to see anything and I just got back the GitLab 404 page.  I tried accessing the site both through http and https but that wasn't the problem.

My next step was to download the artifacts from the GitLab CI and when looking at the files generated, I didn't have any html files, just an index.xml and a site.xml.  A quick google turned up that this was likely due to the theme not being setup.

My problem was that I followed the standard Hugo tutorial practice of cloning a chosen theme under my blog's themes directory, which meant when I pushed my blog to the GitLab repo, they were just checked in as sub-modules.  That means the files for the theme themselves are not checked into my repo and I only had a commit hash associated with each theme diretory.

I use git for all of my professional as well as personal projects, but I've never had to deal with submodules.  Typically people just say "Don't use sub-modules, they're not the solution."

In this particular case, submodules work pretty cleanly for pulling in themes though, so some quick looking around and all I had to do was create a `.gitmodules` file in the root of my blog project that shows the mapping of the theme directory to a repo address that can be pulled from.  I had a few different themes I was trying out, so I put all three in there.

```
[submodule "hugo-multi-bootswatch"]
	path = themes/hugo-multi-bootswatch
	url = https://github.com/mpas/hugo-multi-bootswatch.git
[submodule "hugo-theme-minos"]
	path = themes/hugo-theme-minos
	url = https://github.com/carsonip/hugo-theme-minos
[submodule "robust"]
	path = themes/hugo_theme_robust
	url = https://github.com/dim0627/hugo_theme_robust.git
```

The last step was to add a few steps to the `.gitlab-ci.yml` so that the submodules were initialized and updated before the hugo generation step is run:

```yml
image: publysher/hugo

pages:
  script:
  - git submodule init
  - git submodule update
  - hugo
  artifacts:
    paths:
    - public
  only:
  - master
```

With that pushed to my repo, the next build quickly succeeded and my blog was up and running!

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
