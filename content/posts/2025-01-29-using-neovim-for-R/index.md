+++
title = "A brief guide to using Neovim as an R (and Quarto) development environment"
description = "A top-level overview of how to use Neovim for interactive data analysis with R and Quarto."
date = 2025-01-29
[taxonomies]
tags = ["R", "quarto", "guides"]
+++

For the last year or so I've been using [Neovim](https://neovim.io) as my daily R programming environment after years of using Posit's [RStudio](https://posit.co/products/open-source/rstudio/).
I was mostly just exploring alternative environments for the sake of exploring to begin with, but I found it such an effective editing experience that I've never looked back.

Unlike RStudio, Neovim isn't specialised for R, or for data science more generally, but it's a general purpose tool for coding work and plain text editing. This is great, because it means the user base is huge, and the chances that somebody has already solved whatever problem or pain-point you might run into are high. However, it also means that the information available for how to get up and running with Neovim isn't super tailored to the kinds of features and workflows somebody coming from an RStudio-type environment might expect.

Here, I want to talk a little about my experience with making this shift and provide some advice in the hopes that somebody else might be able to make the transition relatively smoothly. It's a kind of "what I wish I knew when I started out" post.

## Some preliminary notes

Before we get stuck in, let's just do some quick expectation management, both for this post and for switching to Neovim.

### Is Neovim for you?

From my perspective, one of the key reasons for using Neovim is that it offers you the flexibility and control to turn it into the tool you want it to be. Rather than presenting an off-the-shelf experience where the details of how everything works are abstracted away from the user, Neovim offers an initially sparse tool. Through building from that starting point to your own personalised development environment, you learn how the tools you're using every day work, and can cherry pick and tune based on how you would *like* them all to work.

These reasons why I'm drawn to Neovim might be the same reasons that it's not right for you. There is an undeniable learning curve, and if you're not interested in these aspects of your relationship to your development environment, it may not be worth it to you. In the rest of this post, I'm going to assume that you're at least curious.

### Disclaimer 1: I can't speak for all R users

My experience and my advice is based on my own way of working with data in R, which isn't going to be the same as everyone else's. In particular, my workflow is generally built around interactive local work, with lots of data wrangling and exploratory analysis. Write some code, send it to the [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop), see how its output looks, tweak if necessary, repeat. 

I also increasingly use [Quarto](https://quarto.org) notebooks rather than R scripts for a lot of my work, as I usually need to share what I find with non-technical colleagues who are only interested in the results, and Quarto is ideal for this.

My Neovim set-up is built around these two aspects of my workflow, and it might not cover the tooling that others with different needs require. That doesn't mean it's not possible to get what you need in Neovim, only that I can't personally speak to it!

### Disclaimer 2: Don't expect a drop-in replacement for every feature you use now

This isn't a post about how to turn Neovim into a clone of RStudio or VSCode/Positron.
If you want that, it would make more sense to just use those tools and enable their settings for vim keybindings.

Most of what those apps offer can be done in Neovim, but one of the inevitable costs of using a TUI-based setup is that some of their features are out of scope.
In particular, things like the built-in image/plot viewer, the nicely-formatted dataframe viewer, and the web page browser are not easily replicable in Neovim.

For me, this isn't a big deal, because you can just use your window manager to handle those tasks as and when you need them (except for the dataframe viewer - the X window that `View()` invokes in R is hideous and if anyone knows of a better option, I'd love to know).
If you really want them all to be contained within a single application window though, you might not adjust well to Neovim.

## First steps with Neovim

### Vim basics

Neovim is a based on [vim](https://www.vim.org/), with an added focus on making vim's capabilities customisable and extensible through the embedded [Lua](https://www.lua.org) language runtime, as well as providing more built-in tooling (like an LSP client and the Tree-sitter library, more on which [later](#static-code-analysis-lsp-and-tree-sitter)).
In order to get started with Neovim, you need to wrap your head around vim and the idea of _editing modes_ and _motions_ if you haven't encountered these before.
I won't go into that here, because there are far better [resources for learning vim](#useful-links-and-resources).

My advice would be to start with the built-in `vimtutor` tutorial (in Neovim, type `:Tutor` and hit enter), which walks you through the basics.
After that, keep practising by trying to do what you normally do with your code in the vim way for a few days.
Expect to forget key bindings etc., and to find things much slower at first.
But if you stick with it, you'll quickly notice that you have more tools at your fingertips for doing all the most common things you do when editing text files (e.g. changing words and lines, jumping to specific points in the document, repeating changes) naturally and quickly.

### Neovim distributions

Because Neovim (like vim) is rather spartan out of the box, you don't get IDE-like functionality until you start configuring and extending it to do more.
This is a challenge for newcomers, as it takes time to build the knowledge and confidence you need to start personalising the experience based on your preferences.
For this reason, it's common for people to use "Neovim distributions" - which really just means a set of plugins and config files that provide a more ready-to-go (and opinionated) Neovim experience.

I personally recommend avoiding these options at first, and instead starting with the [`kickstart`](https://github.com/nvim-lua/kickstart.nvim) setup, which is designed to achieve the same thing while teaching you how everything works along the way.
My recommendation is to start with `kickstart` (I personally prefer the [modular variant](https://github.com/dam9000/kickstart-modular.nvim) which splits the setup over multiple files) and tweak it as you learn, eventually making it your own.
That's the path I took, and it informs my suggestions throughout this post.

### Neovim's plugin ecosystem

The key to Neovim's power is its plugin ecosystem.
Plugins are written in Lua, and they add new features, customisations and user commands to the base Neovim experience.

If you're following the `kickstart` path, you'll use the [`Lazy.nvim`](https://github.com/folke/lazy.nvim) plugin as your tool for installing and configuring plugins.
The general workflow for this is to create Lua files in a plugin directory within your Neovim config.
In these files, you use Lua tables to provide the location of the repository containing the plugin and the configuration options you want to use, and then `Lazy` will set everything up when you launch Neovim.

## Adding in features

Up until now we've focused on generic Neovim familiarity, but now let's talk about turning Neovim into something useful for doing interactive data analysis work.

The key here is understanding what the useful features that an IDE like RStudio offers are and how they work.
In brief, advanced editors offer static code analysis tooling (which is used for features like auto-indentation, diagnostics and auto-completion), a way of sending code from a source file to a REPL and running it, easy access to a file browser, and other quality of life touches.
Each of these can be added to Neovim.

### Static code analysis: LSP and Tree-sitter

Many of the most useful features that editors offer are based on parsing and understanding your code in order to highlight it, lint it, keep track of document symbols, suggest completions and let you know if something doesn't look right.

Neovim has built-in support for all of this, and it takes two different forms.
First, Neovim contains an [LSP](https://microsoft.github.io/language-server-protocol/) client that can receive information from language servers and report it back to you.
Second, Neovim ships with [Tree-sitter](https://tree-sitter.github.io/), an incremental parsing library that builds an abstract syntax tree (AST) from code.
Together, LSP support and Tree-sitter provide the information that the editor can use to "understand" your code and make suggestions, and many plugins take advantage of this information.

#### Syntax highlighting

Neovim does provide its own default syntax highlighting, but the highlighting based on Tree-sitter's AST is much better.
The way to take advantage of this is to use the [`nvim-treesitter`](https://github.com/nvim-treesitter/nvim-treesitter) plugin to install the R parser and configure it.
Most of the work for setting this up is already handled in `kickstart`, so you just need to make sure the R parser is installed.

#### The R language server

To take advantage of LSP for R, you just need to make sure you have the [`languageserver`](https://github.com/REditorSupport/languageserver/) package installed and then configure it with [`nvim-lspconfig`](https://github.com/neovim/nvim-lspconfig).
The {languageserver} package incorporates several different R packages that handle things like linting and formatting of R code, and covers a lot of LSP capabilities.

In general, the [`mason.nvim`](https://github.com/williamboman/mason.nvim) plugin is a great way to handle the installation of language-specific LSP-related tools, as it provides a centralised registry and will do all the work for you.
However, as R's language server is implemented in R as an R package, I've found that it's best to just install it however you normally install R packages.
The important thing is that you tell `nvim-lspconfig` that [the command to run it is `R --no-echo -e languageserver::run()`](https://github.com/neovim/nvim-lspconfig/blob/master/doc/configs.md#r_language_server).

#### Completion

The primary approach to implementing auto-completion is via the [`nvim-cmp`](https://github.com/hrsh7th/nvim-cmp) plugin.
This plugin draws on various sources such as [LSP](https://github.com/hrsh7th/cmp-nvim-lsp) and the [file system](https://github.com/hrsh7th/cmp-path) to offer a highly customisable list of completion suggestions as you type.
You can also integrate code snippets (e.g. via [`luasnip`](https://github.com/L3MON4D3/LuaSnip)) so that when you confirm a snippet completion suggestion, it will enact a whole snippet insertion.

There does seem to be some buzz around an alternative completion plugin called [`Blink`](https://cmp.saghen.dev/), which claims to be much faster.
However, I haven't tried it yet, so I can't vouch for it.

#### Diagnostics

The [`trouble.nvim`](https://github.com/folke/trouble.nvim) plugin provides a nice way of working with the diagnostics provided by linters and language servers.
Just make sure you customise your [.lintr file](https://lintr.r-lib.org/articles/lintr.html#the--lintr-file) to suit your preferences, otherwise you might be overwhelmed by notes and complaints.

#### Document overview and navigation

LSP and Tree-sitter provide an outline of your document which can be helpful in a few ways.
One is that you can quickly get a sense of the document symbols (i.e. R variables and objects) in your document.
Another is that you can quickly navigate to different points in your document based on these symbols.

For both of these functions, I use the [`aerial.nvim`](https://github.com/stevearc/aerial.nvim) plugin.
It's really useful in Quarto/RMarkdown documents in conjunction with a Markdown language server like [`marksman`](https://github.com/artempyanykh/marksman), because it provides a document outline of headings and subheadings, similar to RStudio's toggleable outline tree.

### Code runners

For me, the key to a good interactive workflow is being able to effortlessly send code to run in the REPL.
In Neovim, there are a few options for replicating this type of behaviour.

The first one that I used for a while is the [`vim-slime`](https://github.com/jpalardy/vim-slime) plugin, which is itself based on an [Emacs feature](https://slime.common-lisp.dev).
That worked pretty well for a while (it's particularly useful for users of terminal multiplexers like [tmux](https://github.com/tmux/tmux)), but it failed to do what I needed often enough that I kept looking for other options.

And then I found [`iron.nvim`](https://github.com/Vigemus/iron.nvim), and I have never looked back.
`Iron` does exactly what I want: it takes some text, and feeds it to a built-in Neovim terminal running an R session where it is executed.
Most importantly, it has customisable keybindings for selecting which text to send, so you can have different shortcuts for sending the current selection, or a line, a paragraph, from the cursor position to the end of the document, and so on.
By integrating vim motions into the "send to REPL" process, `Iron` actually provides a way more powerful version of this key feature than you typically find in other IDEs.

`Iron` has some other nice features, like being able to map file types to REPL programs, so you can send R code to `radian`, Julia code to `julia`, Python code to `ipython`, and so on.

### Quarto-specific features

The key to a smooth Quarto experience in Neovim is the fabulous [`otter.nvim`](https://github.com/jmbuhr/otter.nvim) plugin by Jannik Buhr.
`otter` takes a notebook document and creates hidden buffers (one per embedded language) containing all the code from inside that language's code chunks.
Then, the language server for that language is run on the buffer, and the responses are passed back to the initial notebook document buffer.
It also provides a completion source for tools like `nvim-cmp`.
The upshot of this is that, when you're outside of a code block, you'll get Markdown-based results from LSP-interfacing features like completion, but inside a code block, you'll get responses for the language of the code block.
Thus, when you're working on R code in a .qmd file, and you start typing inside a chunk, the autocompletion suggestions will work as they would in a regular .R file.

Another nice feature is that Tree-sitter knows what a fenced code block looks like in a markdown-based document, which means you can set up your own keybindings for jumping to the next/previous code chunk (I use "[[" and "]]" for this), which makes navigating Quarto notebooks a breeze.
It also means you can highlight your code chunks to keep them visually distinct, and you can use the contents of a code chunk as the text object selection for `Iron` (i.e. you can "run code chunk").

Finally, one feature that makes RStudio a great editor for Quarto documents is that there are commands for "Run all chunks", "Run all chunks above", etc., which execute the code inside code blocks in the REPL.
This is really useful for jumping into a notebook and catching up the REPL's active R session to a certain point in the notebook (for example, to run a data cleaning pipeline before continuing to explore a dataset).
Unfortunately, this being an editor-based feature means that it's not easily replicable in other editors like Neovim where the workflow (e.g. choice of code runner) is more variable between users.
Instead of thinking of running the chunks in a notebook as an editor-specific feature, we can instead think of it as a language-based feature that doesn't depend on editor.
This is what the [{catchup}](https://codeberg.org/pjphd/catchup) package is for.
You call the `catchup()` function from inside R, telling it which notebook to run (and, optionally, which parts to run) and it will do it.
This effectively recreates the "Run all chunks..." family of features, making it accessible in Neovim or any other development environment.

### Other utilities

Finally, here's a quick roundup of some other quality-of-life things that I recommend.

First of all, if you are using R outside of RStudio, you will probably have a nicer time if you use [`radian`](https://github.com/randy3k/radian) as your REPL instead of the base R interpreter.
`radian` takes inspiration from the excellent [Julia REPL](https://docs.julialang.org/en/v1/stdlib/REPL/), and has lots of nice features: it's very configurable, it has built-in syntax highlighting and auto-completion, and it has a super convenient shell mode.
I find the shell mode particularly convenient for Quarto, because the Quarto CLI is just a keystroke away.

Next, you probably want a file browser solution.
There are plenty of popular choices for interacting with the filesystem in Neovim.
If you like a toggleable tree-like sidebar, there's [`nvim-tree`](https://github.com/nvim-tree/nvim-tree.lua) or [`Neo-tree`](https://github.com/nvim-neo-tree/neo-tree.nvim).
I personally use the [`files`](https://github.com/echasnovski/mini.nvim/blob/main/readmes/mini-files.md) module from [`mini.nvim`](https://github.com/echasnovski/mini.nvim), which is a simple dropdown that enables quickly navigating and editing the filesystem.
A similar plugin that is incredibly popular is [`oil.nvim`](https://github.com/stevearc/oil.nvim).

For git integration, there are popular plugins such as [`vim-fugitive`](https://github.com/tpope/vim-fugitive) which may be a good choice for some workflows.
However, I tend to use [`lazygit`](https://github.com/jesseduffield/lazygit) for most git operations, and there is a [`lazygit.nvim`](https://github.com/kdheepak/lazygit.nvim) plugin that integrates it beautifully into Neovim.

I haven't covered debugging here, but there are a lot of tools in this area.
A good place to start is probably [`nvim-dap`](https://github.com/mfussenegger/nvim-dap).

Oh, and I recommend setting up a session manager which can keep track of your Neovim workspace (working directory, active buffers, window layout etc.) and give you a quick way of jumping into projects.
Neovim (via vim) has built-in support for saving and restoring workspace sessions, but plugins like [`neovim-session-manager`](https://github.com/Shatur/neovim-session-manager) help make the process of interacting with that feature more user-friendly.

## Final thoughts

When trying to figure out whether you can do something in Neovim, the general rule of thumb is this:
if your need is related to a general programming/text editing problem, there will certainly be a solution in Neovim (usually via plugins), because it's used by millions of developers from a wide range of languages and domains.
If the need is related to a data science/data analysis problem or is specific to R, there may well be a Neovim solution developed, but it's a little less likely as the user base for those domains just isn't as big.

If you want to take a look at my Neovim config files for inspiration or to try and learn more, they're available [here](https://codeberg.org/pjphd/neovim_config).

If I didn't cover something in this post that you'd like to know more about, let me know and I can update it or maybe do a follow-up post.
Likewise, if you know of a better or different way of doing something than the one I suggested in this post, I'd love to hear about it.
The best way to get in touch is either on the [Fediverse](https://hcommons.social/@petejones) or via [email](mailto:pete@petejon.es).

## Useful links and resources

Here's some useful resources and further reading that should help if you're interested in embarking on your own Neovim journey:

- Vim creator Bram Moolenaar's ["Seven habits of effective text editing"](https://www.moolenaar.net/habits.html) is a common entry point for the vim approach to text editing, as is ["Why, oh WHY, do those #?@! nutheads use vi?"](http://www.viemu.com/a-why-vi-vim.html) by Jon Beltran de Heredia.
- ["Learn Vim Progressively"](https://yannesposito.com/Scratch/en/blog/Learn-Vim-Progressively/) by Yann Esposito offers a useful perspective on building up to being comfortable in vim.
- ["The Only Video You Need to Get Started with Neovim"](https://youtu.be/m8C0Cq9Uv9o?si=sdJfWClsGmHxgqFP) by Neovim contributor TJ DeVries serves as a great introduction to the `kickstart` approach to learning the ropes.
- The ["Understanding Neovim"](https://www.youtube.com/watch?v=87AXw9Quy9U&list=PLx2ksyallYzW4WNYHD9xOFrPRYGlntAft) series by Vhyrro on YouTube is the best entry-level series I've found for learning how Neovim works; I think it's a fantastic resource when learning to customise and configure your setup.
- For Quarto specifically, Jannik Buhr's YouTube [video series on Quarto in Neovim](https://www.youtube.com/watch?v=3sj7clNowlA&list=PLabWm-zCaD1axcMGvf7wFxJz8FZmyHSJ7) covers lots of ground excellently. I learned a lot from seeing Jannik's approach to building a good Quarto experience in Neovim.
- There is a long-running and actively maintained plugin called [`R.nvim`](https://github.com/R-nvim/R.nvim) which bundles lots of R-specific development features into a single package. I don't use it as I prefer the flexibility of managing each of its bundled features separately, but for users just wanting to get up and running with an R workflow in Neovim as quickly as possible, it may be a good option.
