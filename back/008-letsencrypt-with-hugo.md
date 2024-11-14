+++
Categories = ["General"]
date = "2018-02-21T12:00:00+01:00"
title = "Setting up Let's Encrypt with Hugo"
menu = ""
+++

I finally got around to setting up HTTPS with a free TLS certificate for the blog.  As I've mentioned before I mostly use Windows as my daily driver and love using WSL for a native bash/linux experience.  Of course this isn't flawless and sometimes this rears its ugly head.  In particular Let's Encrypts' tool `letsencrypt-auto` won't work under WSL.  It seems to be due to a python package trying to execute code on the stack, which is a security vulnerability that WSL disallows, according to [this github issue](https://github.com/Microsoft/WSL/issues/2553).

I ended up creating a simple virtual machine with Vagrant in order to run let's encrypt and get a certificate generated.  It actually took me longer than I originally expected, given that there are quite a few articles on setting this up.  I followed [this article](https://about.gitlab.com/2016/04/11/tutorial-securing-your-gitlab-pages-with-tls-and-letsencrypt/) that talks specifically about GitLab, where this blog is hosted.  It, however, deals with Jekyll and not Hugo, and it took a few extra non-obvious steps to get things working.

Once you run `letsencrypt-auto certonly --manual -d <domain name>`, it expects you to setup a page with a token to verify that you are the owner of the domain.  I ended up creating a new section by creating a folder under content called 'letsencrypt' and placed a single article 'token.md' under it. In that file I placed the following:

    +++
    url = "<url that letsencrypt-auto is going to look for>"
    layout = "rawtext"
    +++
    <token that letsencrypt-auto wants to find>

I then created a layouts/letsencrypt/rawtext.html file in the base of my blog project with the following

    {{ .Plain }}

The last step, which took me longer than it should have, was to turn off auto-lower casing of urls in site wide config (config.toml):

    ...
    disablePathToLower = true
    ...

With that I was able to generate the certificates that I needed and setup the domain information on Gitlab.

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
