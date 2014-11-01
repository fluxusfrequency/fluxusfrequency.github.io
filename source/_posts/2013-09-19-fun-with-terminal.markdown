---
layout: post
title: "Fun with terminal"
date: 2013-09-19 13:41
---

I love the terminal.

When I first wanted to check out coding, the first exercises I tried were the Javascript ones on [Codecademy](http://www.codecademy.com). I was soon addicted. Something about them felt very familiar in a way that I couldn't put my finger on. I think it may have been the fixed width font. Or maybe it was just the process of interacting with a lot of plain text on the screen instead of a flashy UI (**cough** OS X **cough**). It brought me back to the old days...

That's because my [dad](http://www.clintlewis.com) is a programmer. My grandpa too. Pop brought home a 286 when I was five years old. I think he mostly uses C# these days. He's moved from Windows, to Sun, to some obscure stack I have no comprehension of. *Update - I got a message from Pop that said: "For the record, most of my coding these days is in Java supporting Oracle ADF running in a Weblogic environment. Very corporate."* But when he was just getting started, it was all about MS-DOS. He taught me everything I needed to know; showed me all the right things to type. Commands like cd, dir, mkdir, copy, edit, help. How to search with \*.\*. One of my favorite programs was "simcga", which simulated a cga graphics card so we could play color games on our monochrome monitor (the output was orange). I spent hours playing games like [Tapper](http://www.youtube.com/watch?v=xmqIZMBbMVs). Ctl + Alt + Del was a very important key combination to me. I liked to play Tetris, but I hated to make mistakes, and I thought it was easier to just restart the entire system than to try to figure out how to fix a couple of badly placed blocks. The best program was "helpletmeout.exe", which directed the computer to speak the words: "help, let me out. I'm trapped in this computer" through its very primative built-in speaker. I got pretty effecient at finding my way around the computer and using DOS commands.

When Windows 3.1 was around, I still did a lot of navigating with DOS, but when Windows 95 came out, it became a lot less necessary, and a lot more inconvenient, to access. I remember being pretty bummed out about it at first. But eventually I gave in and started doing all of my file system work through Windows Explorer. It stayed that way until I switched over to Mac in college. Trading Windows XP for OS X, I found myself even further estranged from my DOS roots. Even if I had known that I could access the Unix terminal from OS X, I would have had no idea what to type.

But when I started exploring JS on Codecademy, I got the bug again. I was having fun interacting with the computer through plain text. As I continued to teach myself on their website, and on [Code School](http://www.codeschool.com) and [Treehouse](http://www.teamtreehouse.com), I found out that if I was going to build web applications, I was going to have to learn a little bit of Unix. At first it was intimidating and overwhelming.

Where am I? What? "ls"? That's not intuitive. Where's "dir"? At least "cd" works. Hm, ok, now I'm starting to get the hang of it. "pwd".
"rm". "pushd" and "popd"! Now I'm getting excited. "chown" and "chmod"! I remember kids throwing those ones around in high school when we were wannabe l33t haX0rs!

I had to work at getting comfortable in the bash terminal. But after forcing myself to just try, I actually found myself quite at home. After working on it for several months, I started love, love, love it.

The next logical step was to make it look really cool. So I googled around a bit, and found the [Solarized](http://ethanschoonover.com/solarized) terminal theme. After trying it out, I burned through multiple themes before deciding that it was easier for me to see [IR-Black](https://github.com/jperkins/IR-Black). Next, I thought it would be cool to change the colors in my prompt too. I found [this tutorial](http://wiki.dreamhost.com/Environment_Setup) from Dreamhost on setting up your environment. I added for this line of code to my .bash_profile to get some cool colors:

```bash PS1 Variable
export PS1='\[\033[1;33m\]\u\[\033[1;37m\]@\[\033[1;32m\]\h\[\033[1;37m\]:\[\033[1;31m\]\w \[\033[1;36m\]\$ \[\033[0m\]'
```

Looks like alphabet soup. Or number soup. While I was there, I noticed they had a cool startup graphic. I decided that I wanted a dragon. So I found some ASCII art of a dragon from Joan Stark's awesome Geocities (!!) [homepage](http://www.geocities.com/spunk1111/). Now my terminal boots with this:

```bash Dragon
                                /|   /|
                               / /  / /
                              | |  | |
                              ; /  ; /
                ___  __      / .\_/ ,(   ___
               /  {.' {     / ;;;/ ;;;`"`_.-'
            .--'-./ .-'    / ;;;/ ;;' .-'
         _.-` _  `;;\_,   / ;;;/ .' ,/
     ,-"` ;-= g>=- .; /  / ;;;/ ;',;;|
      \`__/_     ; ';|  | ;;;/.',;;;;'\___,
_    ,=,-V-' __  ;;  _\ /.;/`.`.'` .,:---'
`"=="  |_.-'`  7_;;  | | ;|  .',;;;;;/
              /_,;'  /  \'\ `,;;;;;;|
             {_,;'^ _\  /;;).,,,,,`'\
            {_;;' ^|   /;/`.;;;;;;;-'`
           {_,;'^  _\ //`.,..`_';/
          (__;;  ^  \/`.;;;;;( ',|
          (__,;,^ .'`.;;,.`';;      _.-/
          (__,';,/ ,;'.`;;;,._\  _.'  (
           (__,';|/|;';;.';/` .-' _   /
            (__,-';|; ;;;. |._  `//)_/
             (_/ '- \,'``\ | `'.//
           .-' |'-' '-'';|/ ^   ';
          /'-' | '-' '-' ;`;;.^   `\
         /  '-' \  '-' '/\_, ';,  ^ \
         |'-' .-'\-' '-'|  \_,';,  ^ \
    _ jgs \  /    '.'-' \  //\_,; ^  |
  ,_v\     \ |      \ '-( |;/ \_ ;   |
  v_\ `---_-' \      ;  / |;|  <_; ^ |
  (.-')(`` `'-'      | /  | ;`=';'  ;
   v  </             ||    \ `'`  ^/
                     ||     '-...-'
                     ||
                __  / |_
              v-=_;` __ `)
              v-(_.-'  `v
                 v

```

Once I'd gone this far, I figured I might as well keep going with the customization. So I took another look at the prompt and found other things to add. Here are a couple of useful code bits I added: one to show which [git branch](https://gist.github.com/fabiodan/1649280) I was working from, and one to show which [ruby version](http://rvm.io/workflow/prompt) I was using. Here's what those two modifications look like in my .bash_profile:

```bash Git and RVM in Prompt
# Git branch in prompt.
parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}

#Display ruby version with rvm.
PS1="\$(~/.rvm/bin/rvm-prompt) $PS1"
```

After I did all of that, I had to update my PS1 variable a little more:

```bash PS1 Variable
export PS1="\[\033[33;1m\]\W\[\033[32m\]\$(parse_git_branch)\[\033[00m\] $ "
```

After all of these modifications, here's what my startup screen looks like:

{% img http://s16.postimg.org/y2dy0ulxx/terminal_screenshot.png 'Terminal Screenshot' 'An image of my terminal screen' %}



***UPDATE***

After writing this blog post, I just had my mind blown by my classmates Nathaniel Watts, Rolen Le, and Quentin Tai. They showed me a whole slew of new terminal concepts that completely melted my brain.

On the menu:

* Using [iTerm](http://www.iterm2.com/#/section/home) instead of the Terminal utility that ships with OS X. Apparently it is much better maintained.
* Learning to use Vim. I'm excited to see that fuzzy search and syntax highlighting are easy to get working. I just ran vimtutor from the terminal to get started. Now I know about 0.5% of what there is to know about Vim.
* Learning to use [Tmux](http://robots.thoughtbot.com/post/2641409235/a-tmux-crash-course). Holy crap this looks great for screen sharing.
* Using [Guard](https://github.com/guard/guard) to listen for changes as I try to make tests pass.
* Splitting the screen to have vim editing your code on the left, while guard is running in another pane on the right.

Combining all of these concepts is ridiculously powerful. It means you can have all of your projects open concurrently in separate tmux sessions, with the code open in Vim and Guard listening for changes on each one. To resume work on a project, all you have to do is open the terminal and resume the tmux session for that project.

Sound confusing? I agree! But I'm excited to see the power that's available if I take the time to learn how to use it. Who would have realized that simple text applications could do so much? Regardless of the work involved, one thing is for sure. I am even more in love with the terminal than I was before!
