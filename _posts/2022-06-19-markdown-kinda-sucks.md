---
layout: post
title: Markdown kinda sucks
---
So firstly I do in fact quite like Markdown and generally enjoy writing
documents in it. But similarly to YAML it's simplicity and generally widespread
use has resulted in it being overused and beaten with a large hammer to fit
into places it was never really fit for.

GFM - GitHub Flavored *(Forked)* Markdown
-----------------------------------------
Now somewhat of a standard, they call it a "flavour" - whatever that is
supposed to mean, it's a fork with some extra features added for writing nice
*README* plain-ish text files that render on GitHub projects. It's nice, it's
fine. Use it for your GitHub *README* documents and other GitHub stuff.

It adds mainly `<table>` support to Markdown, also "*tasklist*" which is just a
unordered list with checkboxes instead of bullets, whoop-de-do.

Using it in anger
-----------------
It wont take long before you want to do something super crazy and so out there
you're a lunatic: like aligning text in a single row or cell (YOU ROGUE!).

So OK right, what do you do once you want to do some extremely basic formatting
of your document not supported in markdown but is in HTML? Parsers are pretty
flexible, GitHub and most others render raw HTML tags.

Totally Broken
--------------
Once you've used HTML tags or any other extended crap you've broken any reason
for using markdown in the first place, go open that document in a plaintext
editor like vim or even better just cat it, yeah it's not human readable as
documentation in plaintext anymore is it?

Who cares?
----------
Probably absolutely no one, but you've just wasted your time trying to hack
markdown and breakout HTML into looking how you wanted. The point of writing
docs in markdown should be so they're easily readable as plaintext too.
Otherwise just use a real markup language.

The better way
--------------
Write your doc straight in HTML, or better yet brush up on or learn LaTeX, or
markup for troff. Don't use HTML in the middle of markdown documents, if you do
though that's OK, you should check out `shell_exec` too I think you'd like it.

Markdown is still really great for that quick blog post, reddit post, or any
other simple text article, but stop trying to extend it and beat it into places
it never belonged.
