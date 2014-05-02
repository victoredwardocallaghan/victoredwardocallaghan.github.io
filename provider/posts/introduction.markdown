---
title: Introduction
published: May 4, 2014
excerpt: Nothing
tags: Hakyll, Haskell, Pandoc
---

* toc

Nothing major to say here

## File Structure

My [repositories](https://github.com/victoredwardocallaghan)


Option      Purpose
--------    ---------
build       Generate the entire site
deploy      Deploy the site using a custom deploy procedure

**Build** creates a top-level directory **generated/** with two sub-directories: a directory **cache/** for cached content and a directory **site/** where the compiled site is stored.

**Deploy** puts the compiled site into top-level directory **deploy/** which is git-controlled and force pushes the content to the master branch, effectively deploying (on GitHub).

## Hakyll

For example, in the following Hakyll program:

~~~ {.haskell}
main :: IO ()
main = hakyll $ do
  match "images/*" $ do
      route   idRoute
      compile copyFileCompiler
~~~

`match "images/*"` is a [`Rule`](http://hackage.haskell.org/packages/archive/hakyll/latest/doc/html/Hakyll-Core-Rules.html) that states that the provider directory should match all files matching the glob `images/*`, [`Route`](http://hackage.haskell.org/packages/archive/hakyll/latest/doc/html/Hakyll-Core-Routes.html) them using the `idRoute`, and compile them using the [`Compiler`](http://hackage.haskell.org/packages/archive/hakyll/latest/doc/html/Hakyll-Core-Compiler.html) `copyFileCompiler`.

### Abbreviations {#abbreviations}

### Git Tag

In a [previous post](/posts/commit-tag-for-jekyll/) I talked about a liquid tag I created for Jekyll which inserts the SHA of the commit on which the site was last generated. I have come to like this small feature of my site. It's not some tacky "Powered by blah" footer. It's pretty unobtrusive. It seems unimportant to people who wouldn't understand what it's about, and those who would understand it might immediately recognize its purpose.

<div class="callout">
**Update**: I have stopped including the git commit in the footer of every page. The problem with doing this was that, in order to have every page reflect the new commit, I had to regenerate every page before deploy. This obviously doesn't scale well once more and more pages are added to the site. Instead I have adopted a per-post commit and history link which I believe is a lot more meaningful and meshes perfectly well with generation of pages, i.e. if a post is modified, there'll be a commit made for it and since it was modified it will have to be regenerated anyways. Now I simply include social links in the footer.
</div>

One thing I forgot to update the previous post about was that I ended up switching from using the Rugged git-bindings for Ruby to just using straight up commands and reading their output. The reason for doing this was that, while everything worked perfectly fine on Linux, Rugged had problems building on Windows. It turned out that taking this approach ended up being simpler and had the added benefit of decreasing my dependencies.

The equivalent of a liquid tag in Jekyll would be a field, expressed as a `Context`. For this reason I created the `gitTag` function that takes a desired key, such as `git` --- which would be used as `$git$` in templates --- and returns a `Context` which returns the `String` of formatted HTML. One problem was that to do this I had to use `IO`, so I needed some way to escape the `Compiler` Monad. It turned out that Hakyll already had a function for something like this called `unsafeCompiler`, which it uses for `UnixFilter` for example.

Here's what `gitTag` looks like:

~~~ {.haskell}
gitTag :: String -> Context String
gitTag key = field key $ \_ -> do
  unsafeCompiler $ do
    sha <- readProcess "git" ["log", "-1", "HEAD", "--pretty=format:%H"] []
    message <- readProcess "git" ["log", "-1", "HEAD", "--pretty=format:%s"] []
    return ("<a href=\"https://github.com/blaenk/blaenk.github.io/commit/" ++ sha ++
           "\" title=\"" ++ message ++ "\">" ++ (take 8 sha) ++ "</a>")
~~~

## Conclusion

..

*[AST]: Abstract Syntax Tree
*[EDSL]: Embedded Domain Specific Language
*[GHC]: Glasgow Haskell Compiler
