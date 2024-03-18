+++
title = 'Why I use Vim and why you should too'
date = 2024-03-20T19:42:54+05:30
draft = false
+++

![img](/images/vim_screen.png)

[VIM](https://www.vim.org/) or Vi Improved, is a free and open-source text editor for the terminal written by Bram Moolenaar.
It is a highly powerful and versatile text editor that has managed to gather a devoted following among many people.Known for its efficiency, speed, and extensive customization options, Vim offers a unique modal editing approach that distinguishes it from traditional text editors. 

Although I was already familiar with Vim, but I never thought of actually using it. Thought why should I when there are much modern editors like VSCode and Intellij? Boy was I wrong. 
What pushed me to actually learn Vim motions and incorporate it in my daily editing was none other than, you guessed it right, [ThePrimeagen](https://twitter.com/ThePrimeagen)
When I was writing this article, I thought of including a small section to explain the basic vim motions and commands, but I don't think anything can match the simplicity of the `vimtutor` command.

## What makes Vim different from others

Vim sets itself apart from other text editors through its modal editing system, which offers distinct modes for various tasks, such as normal mode for navigation and command execution, insert mode for text input, and visual mode for selecting text. This unique approach enhances efficiency and control, enabling users to execute commands and navigate documents without relying on the mouse. Additionally, Vim promotes keyboard-centric navigation, replacing traditional arrow keys with the h, j, k, and l keys for movement. By integrating these keys into muscle memory, Vim users can effortlessly traverse text while maintaining optimal hand positioning. This emphasis on keyboard-driven interaction underscores Vim's commitment to enhancing productivity and workflow efficiency.

## My journey of learning Vim motions

### Discovering Vim

When I stumbled upon Prime's [Twitch](https://www.twitch.tv/ThePrimeagen) channel, I was used to using VSCode as my code editor(I still do). Prime uses NeoVim as his editor but I was too intimidated by looking at the amount of configuration needed(Mind you, there was no [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) at that time). So I started off with the `vimtutor` shell command:


![vim_tutor](/images/vimtutor.png)

After getting a little familiar with the basic vim motions, I installed the Vim extension in my VSCode. It did take some time getting used to at first if I'm being honest. But I sticked to it and it gave me wonderful returns on the time saved and how productive I feel while coding, or even making notes in Obsidian.

### The NeoVim saga

![lazy_vim](/images/lazy_vim_screen.png)

After using the Vim extension for 2 years, I felt like I want more speed from my editor which VSCode just couldn't provide. So I referred [Teej](https://www.youtube.com/@teej_dv)'s channel to configure NeoVim from scratch. But being a Windows user, it was quite a pain to go through the tutorial seamlessly due to some dependencies not able to install correctly.

So I tried out NeoVim distros. I tried NVChad, AstroVim, LunarVim but found LazyVim the the most apt to my likes.

While NeoVim provided an unparalleled editing experience, I struggled with setting up Language Server Protocols (LSPs) in Windows—a frustration exacerbated by the effortless extension ecosystem of VSCode. Consequently, I find myself toggling between the two editors, capitalizing on NeoVim's fluidity for certain tasks and relying on VSCode's robust extension support for others.

At this time most of my work revolves around Jupyter Notebooks, and the support for notebooks is still very shaky at best in NeoVim. So have to use VSCode with Vim motions to keep me sane.

## Conclusion

With its intuitive keyboard shortcuts and robust features, Vim empowers users to navigate, edit, and manipulate text with unparalleled efficiency, making it a staple tool in the toolkit of many professionals who demand productivity and precision in their work. Whether you're a seasoned developer seeking maximum productivity or a beginner eager to explore the depths of text manipulation, Vim stands as a cornerstone of the Unix philosophy, embodying simplicity, flexibility, and power.








