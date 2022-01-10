---
title: "Hello World"
layout: post
---

I decided to rip the bandaid off and make a personal website. I need a place to host my portfolio, and I also want to know how to build websites for future projects. 

I started by following this [tutorial](https://www.youtube.com/watch?v=qZsgPgGdOzQ) by Kathryn Schuler. 

There are some good [themes on github](https://github.com/search?q=jekyll+themes):
- https://github.com/niklasbuschmann/contrast
- https://github.com/LeNPaul/academic
- https://github.com/ankitsultana/researcher
- https://github.com/heiswayi/textlog
- https://github.com/nielsenramon/chalk
- https://github.com/rohanchandra/type-theme
- https://github.com/sighingnow/jekyll-gitbook
- https://github.com/huangyz0918/moving

I ended up choosing [contrast](https://github.com/niklasbuschmann/contrast) since it's the closest to what I want and I don't know how to edit the themes yet. 

The initial steps are to fork the repository, rename it to username.github.io, and edit the names / links / etc in `_config.yml`. Be careful to make changes incrementally; a typo may break things. You can see the build-and-deply progress under the Actions tab of the repository. The [documentation](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) is pretty clear that you should be testing locally. 

At this point I installed GitHub Desktop, so I can edit the files locally in Visual Studio Code, and roll back if things die. 

You can find examples of how people edit the content and theme of their websites by browsing the forks of the themes, for example:
- https://github.com/TimothyWeng/TimothyWeng.github.io
- https://heupchurch.github.io/
- https://based-tachikoma.github.io/
- https://github.com/caitlincoffey/caitlincoffey.github.io
- https://github.com/andybridger/andybridger.github.io
- https://github.com/sgul-bioinformatics/sgul-bioinformatics.github.io
- https://github.com/daoxinli/pennchildlanglab.github.io

Each page has some header material defining the title, layout, permalink if necessary, and so on. 

I have yet to get a handle on the [language](https://shopify.github.io/liquid/) that is being used to auto-generate some stuff. 

> Jekyll traverses your site looking for files to process. Any files with front matter are subject to processing. For each of these files, Jekyll makes a variety of data available via Liquid. 

Looks like most of the variables are documented [here](https://jekyllrb.com/docs/variables/). Also, the google results seem reasonably high quality for "jekyll how to do X with liquid".



More links:
- https://www.jasongaylord.com/blog/creating-a-jekyll-theme-from-windows
- https://www.jasongaylord.com/blog/2020/06/18/site-tags-and-sorting-in-jekyll


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
