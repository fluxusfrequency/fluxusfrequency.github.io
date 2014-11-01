---
layout: post
title: "How To Find Big Directories in OS X (Darwin Linux)"
date: 2013-12-30 14:36
---

My Dropbox was full this morning. Since I couldn't afford to buy more space, I needed to clear some room. I set out to find the biggest files and delete them.

Before I got comfortable with the command line, I always downloaded some junky freeware to solve this problem. But today, I figured that there must be a smart way to do this with some arcane Linux incantation, so I did a little research and figured it out. It turned out to be really easy. Here's what to do:

## Step 1

Navigate to the folder you want to search through.

```
$ cd Dropbox
```

## Step 2

Use the `du` (Disk Usage) command to get a list of the files in your current folder (indicated by a `.`) and their sizes. Pass the `-a` flag to search all the files (not just directories), and the `-h` flag to have it display a human readable format.

```
$ du -a -h .
```

## Step 3

Use a pipe to send the results to the `sort` command. Pass `-n` to sort the results numerically, and `-r` to make the biggest files show up at the top of the list.

```
$ du -a -h . | sort -n -r
```

## Step 4

Limit the results to only ten entries so you only have the biggest files.

```
$ du -a -h . | sort -n -r | head -n 10
```

## Step 5

Optionally, dump the whole thing into a text file using the `>` shovel, so you can reference it while you are deleting files.

```
$ du -a -h . | sort -n -r | head -n 10 > file_sizes.txt
```

Open file_sizes.txt in your favorite text editor, and start deleting files in the terminal. To keep the text file up to date, you can hit up arrow a couple of times to get the above command back and run it again after you've deleted a few things.

## Conclusion

Now you can find the biggest files on any part of your hard drive. Just navigate to the folder you want to look at before running the command, or replace the `.` with the path you want to search (e.g. `$ du -a -h ~/Dropbox | sort -n -r | head -n 10 > file_sizes.txt`).

Enjoy!
