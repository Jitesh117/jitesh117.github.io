+++
title = 'Why I use Vim and why you should too'
date = 2024-03-20T07:00:00+05:30
draft = false
cover.image = "/images/vim_screen.png"
ShowToc = true
tags = ["Linux"]
+++

[VIM](https://www.vim.org/) or Vi Improved, is a free and open-source text editor for the terminal written by Bram Moolenaar.
It is a highly powerful and versatile text editor that has managed to gather a devoted following among many people. Known for its efficiency, speed, and extensive customization options, Vim offers a unique modal editing approach that distinguishes it from traditional text editors. 

Although I was familiar with Vim since 2020, I never thought of actually using it. Thought why should I when there are much modern editors like VSCode and Intellij? Boy was I wrong. 
What pushed me to actually learn Vim motions and incorporate it in my daily editing was none other than, you guessed it right, [ThePrimeagen](https://twitter.com/ThePrimeagen).

> But what made me stick to Vim motions was not the absence of shortcuts in other editors, but the ease of doing the mundane things which attracted me.

## What makes Vim different from others

Vim sets itself apart from other text editors such as VSCode through its modal editing system, which offers distinct modes for various tasks.

### But why Modal editing?

- What makes Modal centric editing very useful is the separation of concerns and the elimination of the need to press more than 2/3 keys at once.

- Insert mode ensures that no key combination is pressed accidentally to cause accidental changes to the text/code.

Let's go through some examples to understand this more, shall we

### Going to the bottom most or top most line of code from any line

#### The boring way
To go to the last or the first line of the code, in any other editor, you'll most likely use your mouse and scroll endlessly in case there is 100s or 1000s of lines of code. 
#### The Vim way
Vim makes this super easy and intuitive. Just press `Shift + g` in `Normal` mode (Press Esc to go to `Normal` mode).

To go to the first line of the code from any line, just press `gg` in `Normal` mode (Press Esc to go to `Normal` mode).

After editing those lines, you'd most likely want to go back to the line you were previously working on. Vim has got you covered, just press `Ctrl + o`

### Deleting a whole line of code

#### The boring way
To delete a line of code, one would select the whole line using the mouse and then hit the `backspace` key

#### The Vim way
Just press `dd` in `Normal` mode, easy as pie.

### Copying a function and pasting it somewhere else

#### The boring way
Copy the whole function using the mouse and then Cut and paste

#### The Vim way
Go the the beginning of the function, go to `Normal` mode, press `Shift + V` and then `]]`. The whole function will get selected and then press `d` to delete it and store in the default register.
Then go to the line where you want to paste the code and just press `p`.

There are endless more number of things which can be achieved seamlessly with Vim, but that is out scope for this article. To know more about vim shortcuts, you can follow this [article](https://vim.rtorr.com/).


## My journey of learning Vim motions

### Discovering Vim

When I stumbled upon Prime's [Twitch](https://www.twitch.tv/ThePrimeagen) channel, I was used to using VSCode as my code editor(I still do). Prime uses NeoVim as his editor but I was too intimidated by looking at the amount of configuration needed(Mind you, there was no [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) at that time). So I started off with the `vimtutor` shell command:


![vim_tutor](/images/vimtutor.png)

After getting a little familiar with the basic vim motions, I installed the Vim extension in my VSCode. It did take some time getting used to at first if I'm being honest. But I sticked to it and it gave me wonderful returns on the time saved and how productive I feel while coding, or even making notes in Obsidian.

### The NeoVim saga

![lazy_vim](/images/lazy_vim_screen.png)

After using the Vim extension for 2 years, I felt like I want more speed from my editor which VSCode just couldn't provide. Then I found out about [NeoVim](https://neovim.io/), a modernized and extended version of the Vim text editor, designed to address limitations and add new features while maintaining compatibility with Vim's core functionality and philosophy. I referred [Teej](https://www.youtube.com/@teej_dv)'s channel to configure NeoVim from scratch. But being a Windows user, it was quite a pain to go through the tutorial seamlessly due to some dependencies not able to install correctly.

So I tried out NeoVim distros. I tried NVChad, AstroVim, LunarVim but found LazyVim the the most apt to my likes.

![LazyVim main Screen](/images/lazy_vim.png)

While NeoVim provided an unparalleled editing experience, I struggled with setting up Language Server Protocols (LSPs) in Windows—a frustration solved by the effortless extension ecosystem of VSCode. Consequently, I find myself toggling between the two editors, capitalizing on NeoVim's fluidity for certain tasks and relying on VSCode's robust extension support for others.

At this time most of my work revolves around Jupyter Notebooks, and the support for notebooks is still very shaky at best in NeoVim. So have to use VSCode and Google Colab with Vim motions to keep me sane.

## Conclusion

What I realized after using Vim in both NeoVim and via extensions in VSCode and Intellij, is that although full blown IDE experience from Vim is not for everybody, Vim motions is something which solves a lot of issues for the general public.

Vim motions' capabilities are now so recognised that almost every popular code/text editor has incorporated Vim motions into themselves in the form of extensions/plugins. Even browsers have Vim extensions like Vimium which incorporated the Vim goodness right into your browsing experience.

So all in all, whether you're a seasoned developer seeking maximum productivity or a beginner eager to explore the depths of text manipulation, Vim stands as a cornerstone of the Unix philosophy, embodying simplicity, flexibility, and power.

I hope this article made you curious enough to give Vim a shot and see how it boosts your text editing experience.