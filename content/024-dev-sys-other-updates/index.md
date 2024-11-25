+++
date = 2024-11-25
title = "My Development System and Other Blog Updates"
[taxonomies]
tags = ["Rust", "Zig"]
+++

## Moving to Manjaro

It's been a while since I mentioned changes I've made to my development setup, the last being related to moving to [Kubuntu from Windows 10](@/017-kubuntu/index.md), as well as moving to using [NeoVim as my editor with AstroNVim as the base configuration](@/021-nvim-is-awesome/index.md).  I'm still going strong with NeoVim and loving it as my main editor (including on my phone and tablet via [Termux](https://termux.dev/en/)!).  For the last few years though, I've actually been using [Manjaro Linux](https://manjaro.org/), based on Arch Linux, as my OS and have been very happy with that setup.

I use KDE as my desktop environmet, so apart from using `pamac` instead of `apt` to install packages from the command line, it's relatively transparent that I'm not using an Ubuntu falvor of an OS.

The issue I ran into was that at one point I was using an x.10 release of Ubuntu and held off on updating for a while.  As I've gotten older, I don't want to update a working setup unless I need to.  There's almost never anything in it for me to do a larger update.

Since I had waited too long, the ability to upbdate Kubuntu through the normal `dist-upgrade` channel was turned off, so I was stuck and was going to have to reinstall the OS from scratch.  At that point, my brother mentioned to me that he had been trying various Arch distributions and liked Manjaro.  He's since moved on to EndeavorOS, which I may try sometime.  I gave Manjaro a shot and have had no problems with its rolling updates for over two years.

In the meantime, Steam on Linux has gotten to a point, where almost every title I want to play works on Linux without issues for me.  The one area that's still not perfect is with multiplayer games that use anti-cheat software like Battle Eye ([Rainbox Six Siege, I'm looking at you](https://www.protondb.com/app/359550)).

## Blog Changes

I recently swapped from using Hugo as my static site generator for the blog to using [Zola](https://www.getzola.org/).  My main gripe with Hugo was that I kept getting warnings about themes getting out of date as I would deploy and Hugo on Gitlab would build with a newer version.  I didn't really like how the themes in Hugo were setup, and this would have been the third time I went and hunted down a new theme to use that would be warning free.

Instead, I had heard about Zola and thought I'd give it a try.  The theme engine, shortcode setup and content layout feel far cleaner to me.  That said, I setup my Hugo blog years ago, so maybe things had improved, but since this is just a place for me to publish ideas and articles I don't want to spend much if any time on maintaining my configuration - that takes away from the little time I spend blogging, adding another hurdle.

One big difference in Zola is being able to put my images in the same place as the post.  In Hugo I always had the images under the `static` folder, which kept them far from the posts where they're used.  Just like in code, this ends up being a bad idea.  Again, maybe this was user error, but Zola was setup perfectly right out of the box with good docs.

While working to convert things over, I stumbled on this article: [Why I chose Zola to build this site](https://thedataquarry.com/posts/static-site-zola/), where the author had a lot of the same issues with Hugo that I had.

Converting my blog over to Zola from Hugo and changing over to hosting it on GitHub took about a day and was actually pretty fun.  I also have a feeling this will be a more stable setup, which may actually encourage me to write more.
