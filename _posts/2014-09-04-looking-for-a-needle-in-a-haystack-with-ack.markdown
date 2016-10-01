---
title: Looking For A Needle In A Haystack! (or Using Ack To Improve Your Development
  Workflow)
date: 2014-09-04 12:19:42 Z
categories:
- source
layout: post
---

This post originally appeared on the [Quick Left Blog](http://quickleft.com/blog/looking-for-a-needle-in-a-haystack-or-using-ack-to-improve-your-development-workflow).

## or Using Ack to Improve Your Development Workflow
<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/120/asset.jpg" style="margin:0 auto;display:block;">

## Introduction

Every day when I'm programming, I invariably come to a point where I'm looking for a certain line of code in
my project. Usually, there's a pattern that I saw and want to reuse, and I can't
find it. I could just use my editor's "find in files" feature and look for it, but sometimes I need more fine
grained control. What if I want to find all the lines of code that _don't_
contain a certain phrase? What if I want to search on a Regular
Expression? What if I want to easily save the search results to a file?

When I need more control in finding something, I turn my favorite command line search tool:
[ack](http://beyondgrep.com/).

## Why Ack?

Why should you use `ack`, when your unix distribution comes with `find`, `mdfind`, and
`grep`? Well, because it has these advantages:

- It only searches the stuff you care about. It excludes Git,
  Subversion, binary files, and other irrelevant file types.
- It can search all the files in a certain language, regardless of file
  extension.
- It is a lot easier to remember the command flags to scope your search
  than with other tools.

## Installing Ack

To install Ack, I would suggest using [homebrew](http://brew.sh/). If you
have it installed, just type `brew install ack`.

You can also use `package` on many other Unix distributions, as well as
Macports, OpenBSD and FreeBSD, or just download it with this command:

```
curl http://beyondgrep.com/ack-2.12-single-file > ~/bin/ack && chmod 0755 !#:3
```

## Basic Seach

<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/118/asset.gif" style="margin:0 auto;display:block;">

To do a basic search for a string in a file with ack, the syntax
is as simple as:

```
ack <search-pattern> <directory>
```

This will recursively search for the pattern in specified directory.
I usually just use a simple text search. For example, if you were to do
this:

```
mkdir tmp && cd tmp
echo "Hello, world" > hello.txt
echo "Goodbye, world" > goodbye.txt
echo "Hello, squirrel" > hi.txt
echo "Goodbye, squirrel" > bye.txt
```

Then you could search for the files containing `hello` with:

```
ack hello .
```

The `.` here means search the current directory. This is a recursive
search by default. To turn off recursion, pass the `-n` flag, like this:

```
mkdir child-directory
mv hello.txt child-directory
ack -n hello .
```

Now you'll only see one result, `hi.txt`, because it's the only file
with 'hello' in it that lives in the current directory.

## Sorting

Sometimes it would be nice to sort your search results. Easy enough!

```
ack --sort-files -l
```

## Inverse Search

One of my flags is `-v`, which lets you do a search for all the files
that _don't_ match a given pattern. It comes in pretty handy.

```
ack -v <search-pattern> <directory>
```

Careful, though, because that could be a lot of output. You might want to shovel
it into a file like we did before, or at least pipe it to `less`.

## Searching By File Type

One of my favoritate uses for `ack` is to searching for all the files in a certain language. Ack supports
Ruby, Python, JavaScript, Shell, Clojure, HTML, and a bunch of other
file types. To see a full listing, do:

```
ack --help-types
```

If you want to, you can add new types, change the ones that are already
there, or delete them, with `--type-add`, `--type-set`, and `--type-del`,
respectively.

Assuming you're good with the default types, let's take it for a spin. Want a list of
"all the things" Ruby? Open a Rails project and run this:

```
ack -f --ruby > all-ruby-files.txt
```

Want to find all the Ruby files that call `puts`?

```
ack --ruby puts .
```

Want to find all the files that say `hello`, but _aren't_ Ruby files?

```
ack --type=noruby hello .
```

## Using an `.ackrc` file

If you do a lot of `ack`ing, and you want to set up some ack options
systemwide, or for your specific project, you can define an `.ackrc`
file. To have ack generate one for you, run:

```
ack --create-ackrc > .ackrc
```

Then, if you run a search in the current directory, the settings in the
`.ackrc` will be used.

You can also put your ack options into an `ACK_OPTIONS` environment
variable like so:

```
export ACK_OPTIONS="--nocolor"
```

If you defined some ack options in an `.ackrc` or an environment variable and want to run a search without those options, you can also turn them off with:

```
ack --noenv <search> <directory>
```



## Advanced Searches

If you're diggin' it, here are some other fun things you can do with Ack.

### Case Insenstivite Search

```
ack -i <search-pattern> <directory>
```

### Match Whole Words Only

```
ack -w <search-pattern> <directory>
```

### Only Output the Filenames, Without Highlighted Text

```
ack -l <search-pattern> <directory>
```

```
ack -L <search-pattern> <directory>
```

### Just One Result (AKA "I'm Feeling Lucky")


```
ack -1 <search-pattern> <directory>

```

## Vim Integration (AckVim)

If you're a vim user, you might want to check out the [AckVim Plugin](https://github.com/mileszs/ack.vim).
It lets you run ack inside of vim and see the results in a split window.

Once you've added it with Git or Vundle, it's as easy as typing:

`:Ack [options] {pattern} [{directories}]`

It's pretty nice to have around!

## Ack the Cat, Cathy and Bar

Well, you've made it this far, so here's some candy for you. Run these:

```
ack --thpppt
ack --cathy
ack --bar
```

## Goodbye!

Sometimes it can be like looking for a needle in
a haystack trying to find what you need in your project.

<img src="//quickleft-production.s3.amazonaws.com/uploads/asset/attachment/119/asset.gif" style="margin:0 auto;display:block;">

Seriously, though, I hope this little tour of the small command line
tool `ack` improves your programming experience. Cheers!
