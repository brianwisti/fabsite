fabsite.py
===========

Version
: 2022.1001-alpha

Take full control of your static website/blog generation by writing your
own simple, lightweight, and magic-free static site generator in
Python. That's right! Reinvent the wheel!

[![MIT License][LICENSE-BADGE]](LICENSE.md)

[LICENSE-BADGE]: https://img.shields.io/badge/license-MIT-blue.svg


Contents
--------

* [Introduction](#introduction)
* [But Why?](#but-why)
* [Get Started](#get-started)
* [The Code](#the-code)
* [Layout](#layout)
* [Content](#content)
* [FAQ](#faq)
* [Credits](#credits)
* [License](#license)
* [Support](#support)


Introduction
------------

This repository started with the [makesite.py][makesite] source. I took
[Sunaina Pai][sunaina-pai] up on the invitation to reinvent the wheel using
that project as the base. It's quickly becoming thoroughly unrecognizable.

[makesite]: https://github.com/sunainapai/makesite
[sunaina-pai]: https://github.com/sunainapai

So go ahead, fork this repository, replace the [content](content) with
your own, and generate your static website. It's that simple!

You are [free](LICENSE.md) to copy, use, and modify this project for
your blog or website, so go ahead and fork this repository and make it
your own project. Change the [layout](layout) if you wish to, improve
the [stylesheet](static/css/style.css) to suit your taste, enhance
[fabsite.py](fabsite.py) if you need to, and develop your website/blog
just the way you want it.

Get Started
-----------

This section provides some quick steps to get you off the ground as
quickly as possible.

 1. Install the dependencies

        python -m pip install --upgrade -r requirements.txt

 2. For a quick demo on your local system, just enter this command:

        fab serve

    Then visit http://localhost:8000/.

    Note: My version of [fabsite.py](fabsite.py) assumes Python 3.9.x

 3. For an Internet-facing website, you would be hosting the static
    website/blog on a hosting service and/or with a web server such as
    Apache HTTP Server, Nginx, etc. You probably only need to generate
    the static files and know where the static files are and move them
    to your hosting location.

    Enter this command to generate your website:

        fab site

    The `_site` directory contains the entire generated website. The
    content of this directory may be copied to your website hosting
    location.


Noteworthy Deviations
---------------------

`makesite.py` is a very straightforward, practical static site generator.
My version is — not. But I try to keep it organized. Here are some of the
changes I made.

- Run tasks with [Fabric][fabric] instead of Make
- Run tests with [Pytest][pytest] instead of unittest.

[fabric]: https://fabfile.org
[pytest]: htts://pytest.org

The Code
--------

Now that you know how to generate the static website that comes with
this project, it is time to see what [makesite.py](makesite.py) does.
You probably don't really need to read the entire section. The source
code is pretty self-explanatory but just in case, you need a detailed
overview of what it does, here are the details:

 1. The `main()` function is the starting point of website generation.
    It calls the other functions necessary to get the website generation
    done.

 2. First it creates a fresh new `_site` directory from scratch. All
    files in the [static directory](static) are copied to this
    directory. Later the static website is generated and written to this
    directory.

 3. Then it creates a `params` dictionary with some default parameters.
    This dictionary is passed around to other functions. These other
    functions would pick values from this dictionary to populate
    placeholders in the layout template files.

    Let us take the `subtitle` parameter for example. It is set
    to our example website's fictitious brand name: "Lorem Ipsum". We
    want each page to include this brand name as a suffix in the title.
    For example, the [about page](https://tmug.github.io/makesite-demo/about/)
    has "About - Lorem Ipsum" in its title. Now take a look at the
    [page layout template](layout/page.html) that is used as the layout
    for all pages in the static website. This layout file uses the
    `{{ subtitle }}` syntax to denote that it is a placeholder that
    should be populated while rendering the template.

    Another interesting thing to note is that a content file can
    override these parameters by defining its own parameters in the
    content header. For example, take a look at the content file for
    the [home page](content/_index.html). In its content header, i.e.,
    the HTML comments at the top with key-value pairs, it defines a new
    parameter named `title` and overrides the `subtitle` parameter.

    We will discuss the syntax for placeholders and content headers
    later. It is quite simple.

 4. It then loads all the layout templates. There are 6 of them in this
    project.

      - [layout/page.html](layout/page.html): It contains the base
        template that applies to all pages. It begins with
        `<!DOCTYPE html>` and `<html>`, and ends with `</html>`. The
        `{{ content }}` placeholder in this template is replaced with
        the actual content of the page. For example, for the about page,
        the `{{ content }}` placeholder is replaced with the the entire
        content from [content/about.html](content/about.html). This is
        done with the `make_pages()` calls further down in the code.

      - [layout/post.html](layout/post.html): It contains the template
        for the blog posts. Note that it does not begin with `<!DOCTYPE
        html>` and does not contain the `<html>` and `</html>` tags.
        This is not a complete standalone template. This template
        defines only a small portion of the blog post pages that are
        specific to blog posts.  It contains the HTML code and the
        placeholders to display the title, publication date, and author
        of blog posts.

        This template must be combined with the
        [page layout template](layout/page.html) to create the final
        standalone template. To do so, we replace the `{{ content }}`
        placeholder in the [page layout template](layout/page.html) with
        the HTML code in the [post layout template](layout/post.html) to
        get a final standalone template. This is done with the
        `render()` calls further down in the code.

        The resulting standalone template still has a `{{ content }}`
        placeholder from the [post layout template](layout/post.html)
        template.  This `{{ content }}` placeholder is then replaced
        with the actual content from the [blog posts](content/blog).

      - [layout/list.html](layout/list.html): It contains the template
        for the blog listing page, the page that lists all the posts in
        a blog in reverse chronological order. This template does not do
        much except provide a title at the top and an RSS link at the
        bottom.  The `{{ content }}` placeholder is populated with the
        list of blog posts in reverse chronological order.

        Just like the [post layout template](layout/post.html) , this
        template must be combined with the
        [page layout template](layout/page.html) to arrive at the final
        standalone template.

      - [layout/item.html](layout/item.html): It contains the template
        for each blog post item in the blog listing page. The
        `make_list()` function renders each blog post item with this
        template and inserts them into the
        [list layout template](layout/list.html) to create the blog
        listing page.

      - [layout/feed.xml](layout/feed.xml): It contains the XML template
        for RSS feeds. The `{{ content }}` placeholder is populated with
        the list of feed items.

      - [layout/item.xml](layout/item.xml): It contains the XML template for
        each blog post item to be included in the RSS feed. The
        `make_list()` function renders each blog post item with this
        template and inserts them into the
        [layout/feed.xml](layout/feed.xml) template to create the
        complete RSS feed.

 5. After loading all the layout templates, it makes a `render()` call
    to combine the [post layout template](layout/post.html) with the
    [page layout template](layout/page.html) to form the final
    standalone post template.

    Similarly, it combines the [list layout template](layout/list.html)
    template with the [page layout template](layout/page.html) to form
    the final list template.

 6. Then it makes two `make_pages()` calls to render the home page and a
    couple of other site pages: the [contact page](content/contact.html)
    and the [about page](content/about.html).

 7. Then it makes two more `make_pages()` calls to render two blogs: one
    that is named simply [blog](content/blog) and another that is named
    [news](content/news).

    Note that the `make_pages()` call accepts three positional
    arguments:

      - Path to content source files provided as a glob pattern.
      - Output path template as a string.
      - Layout template code as a string.

    These three positional arguments are then followed by keyword
    arguments. These keyword arguments are used as template parameters
    in the output path template and the layout template to replace the
    placeholders with their corresponding values.

    As described in point 2 above, a content file can override these
    parameters in its content header.

 8. Then it makes two `make_list()` calls to render the blog listing
    pages for the two blogs. These calls are very similar to the
    `make_pages()` calls. There are only two things that are different
    about the `make_list()` calls:

      - There is no point in reading the same blog posts again that were
        read by `make_pages()`, so instead of passing the path to
        content source files, we feed a chronologically reverse-sorted
        index of blog posts returned by `make_pages()` to `make_list()`.
      - There is an additional argument to pass the
        [item layout template](layout/item.html) as a string.

 9. Finally it makes two more `make_list()` calls to generate the RSS
    feeds for the two blogs. There is nothing different about these
    calls than the previous ones except that we use the feed XML
    templates here to generate RSS feeds.

To recap quickly, we create a `_site` directory to write the static site
generated, define some default parameters, load all the layout
templates, and then call `make_pages()` to render pages and blog posts
with these templates, call `make_list()` to render blog listing pages
and RSS feeds. That's all!

Take a look at how the `make_pages()` and `make_list()` functions are
implemented. They are very simple with less than 20 lines of code each.
Once you are comfortable with this code, you can begin modifying it to
add more blogs or reduce them. For example, you probably don't need a
news blog, so you may delete the `make_pages()` and `make_list()` calls
for `'news'` along with its content at [content/news](content/news).


Layout
------

In this project, the layout template files are located in the [layout
directory](layout). But they don't necessarily have to be there. You can
place the layout files wherever you want and update
[makesite.py](makesite.py) accordingly.

The source code of [makesite.py](makesite.py) that comes with this
project understands the notion of placeholders in the layout templates.
The template placeholders have the following syntax:

    {{ <key> }}

Any whitespace before `{{`, around `<key>`, and after `}}` is ignored.
The `<key>` should be a valid Python identifier. Here is an example of
template placeholder:

    {{ title }}

This is a very simple template mechanism that is implemented already in
the [makesite.py](makesite.py). For a simple website or blog, this
should be sufficient. If you need a more sophisticated template engine
such as [Jinja2](http://jinja.pocoo.org/) or
[Cheetah](https://pythonhosted.org/Cheetah/), you need to modify
[makesite.py](makesite.py) to add support for it.


Content
-------

In this project, the content files are located in the [content
directory](content). Most of the content files are written in HTML.
However, the content files for the blog named [blog](content/blog) are
written in Markdown.

The notion of headers in the content files is supported by
[makesite.py](makesite.py). Each content file may begin with one or more
consecutive HTML comments that contain headers. Each header has the
following syntax:

    <!-- <key>: <value> -->

Any whitespace before, after, and around the `<!--`, `<key>`, `:`,
`<value>`, and `-->` tokens are ignored. Here are some example headers:

    <!-- title: About -->
    <!-- subtitle: Lorem Ipsum -->
    <!-- author: Admin -->

It looks for the headers at the top of every content file. As soon as
some non-header text is encountered, the rest of the content from that
point is not checked for headers.

By default, placeholders in content files are not populated during
rendering. This behaviour is chosen so that you can write content freely
without having to worry about makesite interfering with the content,
i.e., you can write something like `{{ title }}` in the content and
makesite would leave it intact by default.

However if you do want to populate the placeholders in a content file,
you need to specify a parameter named `render` with value of `yes`. This
can be done in two ways:

  - Specify the parameter in a header in the content file in the
    following manner:

        <!-- render: yes -->

  - Specify the parameter as a keyword argument in `make_pages` call.
    For example:

        blog_posts = make_pages('content/blog/*.md',
                                '_site/blog/{{ slug }}/index.html',
                                post_layout, blog='blog', render='yes',
                                **params)

FAQ
---

Here are some frequently asked questions along with answers to them:

 1. Can you add feature X to this project?

    I do not have any plans to add new features to this project. It is
    intended to be as minimal and as simple as reasonably possible. This
    project is meant to be a quick-starter-kit for developers who want
    to develop their own static site generators. Someone who needs more
    features is free to fork this project repository and customize the
    project as per their needs in their own fork.

 2. Can you add support for Jinja templates, YAML front matter, etc.?

    I will not add or accept support for Jinja templates, YAML front
    matter, etc. in this project. However, you can do so in your fork.
    The reasons are explained in the first point.

 3. Do you accept any new features from the contributors?

    I do not accept any new features in this project. The reasons are
    explained in the first point.

 4. Do you accept bug fixes and improvements?

    Yes, I accept bug fixes and minor improvements that do not increase
    the scope and complexity of this project.

 5. Are there any contribution guidelines?

    Yes, please see [CONTRIBUTING.md](CONTRIBUTING.md).

 6. How do I add my own copyright notice to the source code without
    violating the terms of license while customizing this project in my
    own fork?

    This project is released under the terms of the MIT license. One of
    the terms of the license is that the original copyright notice and
    the license text must be preserved. However, at the same time, when
    you edit and customize this project in your own fork, you hold the
    copyright to your changes. To fulfill both conditions, please add
    your own copyright notice above the original copyright notice and
    clarify that your software is a derivative of the original.

    Here is an example of such a notice where a person named J. Doe
    wants to reserve all rights to their changes:

        # Copyright (c) 2018-2019 J. Doe
        # All rights reserved

        # This software is a derivative of the original makesite.py.
        # The license text of the original makesite.py is included below.

    Anything similar to the above notice or something to this effect is
    sufficient.


Credits
-------

Thanks to:

  - [Susam Pal](https://github.com/susam) for the initial documentation
    and the initial unit tests.
  - [Keith Gaughan](https://github.com/kgaughan) for an improved
    single-pass rendering of templates.


License
-------

This is free and open source software. You can use, copy, modify,
merge, publish, distribute, sublicense, and/or sell copies of it,
under the terms of the [MIT License](LICENSE.md).

This software is provided "AS IS", WITHOUT WARRANTY OF ANY KIND,
express or implied. See the [MIT License](LICENSE.md) for details.


Support
-------

To report bugs, suggest improvements, or ask questions, please visit
<https://github.com/sunainapai/makesite/issues>.
