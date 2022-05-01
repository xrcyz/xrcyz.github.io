---
title: "Hello World"
layout: post
---

I decided to rip the bandaid off and make a personal website. I need a place to host my portfolio, and I also want to know how to build websites for future projects. 

> TLDR follow [this tutorial]([this](https://www.taniarascia.com/make-a-static-website-with-jekyll/))

I started by following this [tutorial](https://www.youtube.com/watch?v=qZsgPgGdOzQ) by Kathryn Schuler. 

There are some good [themes on github](https://github.com/search?q=jekyll+themes):
- [contrast](https://github.com/niklasbuschmann/contrast)
- [academic](https://github.com/LeNPaul/academic)
- [researcher](https://github.com/ankitsultana/researcher)
- [textlog](https://github.com/heiswayi/textlog)
- [chalk](https://github.com/nielsenramon/chalk)
- [type](https://github.com/rohanchandra/type-theme)
- [jekyll-gitbook](https://github.com/sighingnow/jekyll-gitbook)
- [moving](https://github.com/huangyz0918/moving)

I ended up choosing [contrast](https://github.com/niklasbuschmann/contrast) since it's the closest to what I want and I don't know how to edit the themes yet. 

The initial steps are to fork the repository, rename it to username.github.io, and edit the names / links / etc in `_config.yml`. Be careful to make changes incrementally; a typo may break things. You can see the build-and-deply progress under the Actions tab of the repository. Note that sometimes a successful build still has a delay before showing changes, some changes didn't reload for another 10 minutes. The [documentation](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) is pretty clear that you should be testing locally. 

At this point I installed GitHub Desktop, so I can edit the files locally in Visual Studio Code, and roll back if things die. 

You can find examples of how people edit the content and theme of their websites by browsing the forks of the themes, for example:
- [https://github.com/daoxinli/pennchildlanglab.github.io](https://github.com/daoxinli/pennchildlanglab.github.io)
- [https://github.com/sgul-bioinformatics/sgul-bioinformatics.github.io](https://github.com/sgul-bioinformatics/sgul-bioinformatics.github.io)
- [https://github.com/TimothyWeng/TimothyWeng.github.io](https://github.com/TimothyWeng/TimothyWeng.github.io)
- [https://github.com/heupchurch/heupchurch.github.io/](https://github.com/heupchurch/heupchurch.github.io/)
- [https://github.com/based-tachikoma/based-tachikoma.github.io/](https://github.com/based-tachikoma/based-tachikoma.github.io/)
- [https://github.com/caitlincoffey/caitlincoffey.github.io](https://github.com/caitlincoffey/caitlincoffey.github.io)
- [https://github.com/andybridger/andybridger.github.io](https://github.com/andybridger/andybridger.github.io)

I have yet to get a handle on the [language](https://shopify.github.io/liquid/) that is being used to auto-generate some stuff. 

> Jekyll traverses your site looking for files to process. Any files with front matter are subject to processing. For each of these files, Jekyll makes a variety of data available via Liquid. 

Looks like most of the variables are documented [here](https://jekyllrb.com/docs/variables/). Also, the google results seem reasonably high quality for "jekyll how to do X with liquid".

Take this [example](https://raw.githubusercontent.com/daoxinli/pennchildlanglab.github.io/master/people.md) that lays out a grid on the page: 

{% raw %}
```html
<ul class="list-unstyled list-inline text-center">

{% for author in site.authors %}
<li>
        <figure class="figure">
                <img src='../assets/images/{{ author.short_name }}.png' alt='{{ author.short_name }}' /> 
                <figcaption><strong><a href="{{ author.url }}">{{ author.name }}</a></strong>
                <br> {{ author.position }} </figcaption>
        </figure> 
</li>
{% endfor %}
</ul>
```
{% endraw %}

The `ul` starts a HTML unordered list, `li` is an item in the list. The list `site.authors` is defined in `_config.yml` under the `collections` header:

```md
# collections
collections:
  authors:
    output: true
```

Then there is a folder called `_authors` containing the individual `author1.md` files, with the `author.xyz` variables defined in the header block. 

The [academic](https://github.com/LeNPaul/academic) theme does the same thing with its [portfolio](https://github.com/academicpages/academicpages.github.io/blob/25c30de2b4ce3e3f23559384699bb4b9865d6473/_pages/portfolio.html).

And now that I'm looking for it, the `layout` types in the header blocks get mapped to html files in the `_layouts` folder. 

A bit of googling takes me to [this](https://www.taniarascia.com/make-a-static-website-with-jekyll/) awesome tutorial by Tania Rascia, which fills in most of the gaps.

More links:
- [set link preview image?](https://agustinus.kristia.de/techblog/2016/08/15/jekyll-fb-share/)
- https://carpentries-incubator.github.io/jekyll-pages-novice/arrays/index.html
- https://www.jasongaylord.com/blog/creating-a-jekyll-theme-from-windows
- https://www.jasongaylord.com/blog/2020/06/18/site-tags-and-sorting-in-jekyll


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
