+++
tags = ["Vim"]
date = "2023-04-18T12:00:00-08:00"
title = "Neovim with a Little Bit of Config is Amazing"
+++

I've recently switched from using Visual Studio Code as my main editor of choice to using Neovim with the awesome [Astronvim](https://astronvim.com/) configuration and some of my own tweaks on top. It ends up functioning as a great development environment with code hints, debugging, file finder and at the same time uses much less memory. It even has one killer feature over VS Code.

<!--more-->

## Background

I had learned and used Vim a long time ago in college, when connecting over ssh to the school computers to do homework. I never felt quite comfortable using it as my main environment, but for the time (this was about 22 years ago), it worked well enough. I ended up preferring Visual Studio proper once I got out into the working world as I was doing C# programming at the time. When I moved to C++, I was still primarily working on Windows so Visual Studio Pro was still my go to.

Once Visual Studio Code came out, I jumped over to using it back around 2017 and have been using it ever since. It's a great editor and the extensions can be really powerful. I have however run into issues over time with memory load being too high, the electron base using GPU rendering draining more battery than simple text editing should and most recently lots of crashes from the Rust extension, apparently running out of memory in the background.

## Another look at (Neo)Vim

I occasionally like to look for new tools to make my workflow more efficient. Changing over to the [Fish shell](https://fishshell.com), using [lcd](https://github.com/lsd-rs/lsd) as an `ls` replacement or [bat](https://github.com/sharkdp/bat) instead of `cat` have been nice improvements. I stumbled onto the [NvChad](https://github.com/NvChad/NvChad) configuration about six months ago and played with it for a bit - it was really nice, but I just didn't take the time to dial everything in the way I liked. I kept it in the back of my mind though as it looked like a great tool for ssh'ing into a remote server or VM to work on.

With VS Code starting to give me occasional issues, I looked at NvChad again, but my config had somehow gotten out of date and was broken. Some searching around mentioned Astronvim and I decided to give it a shot. They had just converted to v3.0, using [lazy.nvim](https://github.com/folke/lazy.nvim) as the plugin loader and out of the box, the experience is _really_ good.

Within ten minutes, I had syntax highlighting and code hints for Rust and Elixir working, was able to pop open the file explorer, and was really liking the floating terminal window.

![Nvim Editing this Article](/021-nvim-is-awesome/nvim_writing_article.png)

## Killer feature

I had seen the [Telescope](https://github.com/nvim-telescope/telescope.nvim) plugin with NvChad before and really liked the fuzzy file finding, similar to VS Code's `Ctrl+P` feature. What I found this time was the Find Buffers version, default keymapping `<Leader>+fb` but more interestingly the Live Grep feature.

![Nvim using Telescope](/021-nvim-is-awesome/nvim_telescope.png)

With Live Grep, I can start a `Find in Files`-style search, but easily move through the matches and see the context of the find. It also happens nearly instantly. When I've tried Find in Files with VS Code, it usually takes more than a couple of seconds, and the UX just feels slower.

For me, given some of the projects I've been working on which can span multiple repositories - so that simple project based symbol finding is not as useful - this was the feature that made me think "Hmm, maybe I could replace VS Code with this now..."

After a number of little tweaks here and there to get the keybindings setup the way I like, I'm finding it more and more useful. In particular I found, thanks to a colleague, the [vimwiki](https://github.com/vimwiki/vimwiki) plugin, which has been the first major change to my note taking workflow in about a decade. I highly recommend trying it out (with my tweaks below of course :smile: ).

## My Configuration

All of my tweaks are in a single user configuration file, which lives under `~/.config/nvim/lua/user/init.lua`. I've placed my version [here on Github](https://github.com/srjilarious/configs/blob/master/nvim_config/init.lua) (Obviously, that version may be more up to date than this article as I find more things to improve).

### Line Moving

I really like the feature in VS Code of being able to move a line, or a set of lines with a control key and the arrow keys. Being vim, I've instead opted for the `j`/`k` style movement keys. It turned out that getting movement working in visual mode was being a pain, and some interactive help with ChatGPT sent me on a working path using lua functions instead:

```lua
-- Keymappings for moving lines up/down
vim.api.nvim_set_keymap('i', '<M-k>', '<Esc>:m .-2<CR>gi', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<M-j>', '<Esc>:m .+1<CR>gi', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<M-k>', '<Esc>:m .-2<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<M-j>', '<Esc>:m .+1<CR>', { noremap = true, silent = true })

vim.api.nvim_set_keymap('v', '<M-k>', "<Esc>:lua MoveBlockUp()<CR>", { noremap = true, silent = true })
vim.api.nvim_set_keymap('v', '<M-j>', "<Esc>:lua MoveBlockDown()<CR>", { noremap = true, silent = true })

local function move_block_up()
    local start_line = vim.fn.line("'<")
    local end_line = vim.fn.line("'>")
    if start_line == 1 then return end
    vim.cmd("silent! normal! " .. (start_line-1) .. "Gdd")
    vim.cmd("silent! normal! " .. (end_line-1) .. "Gp")
    vim.cmd("silent! normal! " .. (start_line-1) .. "GV" .. (end_line-1) .. "G")
end

local function move_block_down()
    local start_line = vim.fn.line("'<")
    local end_line = vim.fn.line("'>")
    if end_line == vim.fn.line("$") then return end
    vim.cmd("silent! normal! " .. (end_line+1) .. "Gdd")
    vim.cmd("silent! normal! " .. start_line-1 .. "Gp")
    vim.cmd("silent! normal! " .. (start_line+1) .. "GV" .. (end_line+1) .. "G")
end

_G.MoveBlockUp = move_block_up
_G.MoveBlockDown = move_block_down

```

### Line Duplicating

Similar to moving lines, I often find my self wanting to duplicate the current line up or down, or a selection of lines. I've grown accustomed to that workflow in VS Code, and it's a nice feature to have in Neovim as well. The following keybindings implement that feature:

```lua
-- Duplicate lines keymappings
vim.api.nvim_set_keymap('i', '<M-S-k>', '<Esc>:t .-1<CR>gi', { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<M-S-j>', '<Esc>:t .<CR>gi', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<M-S-k>', '<Esc>:t .-1<CR>', { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<M-S-j>', '<Esc>:t .<CR>', { noremap = true, silent = true })

vim.api.nvim_set_keymap('v', '<M-S-k>', "<Esc>:lua CopyBlockUp()<CR>", { noremap = true, silent = true })
vim.api.nvim_set_keymap('v', '<M-S-j>', "<Esc>:lua CopyBlockDown()<CR>", { noremap = true, silent = true })

local function copy_block_up()
    local start_line = vim.fn.line("'<")
    local end_line = vim.fn.line("'>")
    local lines_to_duplicate = vim.api.nvim_buf_get_lines(0, start_line - 1, end_line, false)
    vim.api.nvim_buf_set_lines(0, end_line, end_line, false, lines_to_duplicate)
    local new_end_line = end_line + (end_line-start_line) + 1
    vim.cmd("silent! normal! " .. end_line+1 .. "GV" .. new_end_line .. "G")
end

local function copy_block_down()
    local start_line = vim.fn.line("'<")
    local end_line = vim.fn.line("'>")

    local lines_to_duplicate = vim.api.nvim_buf_get_lines(0, start_line - 1, end_line, false)
    vim.api.nvim_buf_set_lines(0, end_line, end_line, false, lines_to_duplicate)
    vim.cmd("silent! normal! " .. start_line .. "GV" .. end_line .. "G")

end

_G.CopyBlockUp = copy_block_up
_G.CopyBlockDown = copy_block_down
```

### Key bindings for telescope

I've gotten used to `Ctrl+P` looking for files, so I bound that to the Telescope plugin and also added `Ctrl+B` to search through the current buffer's list. I use Live Grep less often, so the default `<Leader>fw` is not too akward to rely on.

```lua
-- Telescope find files
vim.api.nvim_set_keymap('n', '<C-p>', "<cmd>Telescope find_files<cr>", { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-p>', "<Esc><cmd>Telescope find_files<cr><gi>", { noremap = true, silent = true })

-- Telescope find buffer mappings
vim.api.nvim_set_keymap('n', '<C-b>', "<cmd>Telescope buffers<cr>", { noremap = true, silent = true })
vim.api.nvim_set_keymap('i', '<C-b>', "<Esc><cmd>Telescope buffers<cr><gi>", { noremap = true, silent = true })

```

### Buffer switching

By default, Astronvim has `<Leader>+[b` and `<Leaader>+]b` as the bindings for switching between buffers (think open tabs). I ended up finding this a bit annoying to use in practice, so I changed my Terminal editor to use `Win+Tab` and `Win+Shift+Tab` to switch between terminal tabs leaving `Ctrl+Tab` and `Ctrl+Shift+Tab` open for switching buffers:

```lua
-- Switch to the buffers using Ctrl+(Shift+)Tab
vim.api.nvim_set_keymap('n', '<C-Tab>', ':bnext<CR>', {noremap = true, silent = true})
vim.api.nvim_set_keymap('i', '<C-Tab>', '<Esc>:bnext<CR>gi', {noremap = true, silent = true})
vim.api.nvim_set_keymap('n', '<C-S-Tab>', ':bprevious<CR>', {noremap = true, silent = true})
vim.api.nvim_set_keymap('i', '<C-S-Tab>', '<Esc>:bprevious<CR>gi', {noremap = true, silent = true})
```

### Entering the Correct Directory on Startup

One thing I noticed was that if I start `nvim` from my home directory, giving it the directory to open, the file explorer would work, but neovim would still be in my home directory as its current working directory. This ended up messing with things like Telescope where I wanted to search starting from the directory I had opened. I had to do a similar tweak so that the floating terminal (bound to `F7` by the way) would start in the correct directory as well:

```lua
-- Enter the current directory when vim starts
-- % curr file, :p full path, :h get dir
local file_path = vim.fn.expand("%p")
local is_dir = vim.fn.isdirectory(file_path)
if is_dir == 1 then
  vim.api.nvim_create_autocmd("VimEnter", {pattern = "*", command = "silent! cd %:p"})
  vim.api.nvim_create_autocmd("VimEnter", {pattern = "*", command = "TermExec open=0 cmd='cd %:p && clear'"})
else
  vim.api.nvim_create_autocmd("VimEnter", {pattern = "*", command = "silent! cd %:p:h"})
  vim.api.nvim_create_autocmd("VimEnter", {pattern = "*", command = "TermExec open=0 cmd='cd %:p:h && clear'"})
end

```

### Visualizing whitespace

I like being able to see whitespace, trailing and otherwise, and also wanted word wrapping to show a line wrap and indent a little bit. The following tweaks got that working:

```lua
vim.opt.list = true
vim.opt.listchars:append({space = "·", lead = "·", tab = "» ", trail = "·"}) --, trail = '·', setbreak = "···", space = '·' }
vim.opt.showbreak = "↪  "
```

### The Rest of the Plugin Configuration

At the bottom of my user `init.lua` file, I include a number of AstroNvim community plugins and packs: rust, markdown, various colorschemes. I also modified the starting header, added mode text to the status line,

The community plugins and colorschemes portion looks like this:

```lua
return {
  plugins = {
    "AstroNvim/astrocommunity",
    -- ...
    { import = "astrocommunity.pack.rust" },
    { import = "astrocommunity.pack.markdown" },
    { import = "astrocommunity.colorscheme.kanagawa", lazy=false},
    { import = "astrocommunity.colorscheme.catppuccin", lazy = false},
    { import = "astrocommunity.colorscheme.oxocarbon", lazy=false},
    { import = "astrocommunity.colorscheme.nightfox", lazy=false},
    { import = "astrocommunity.motion.leap-nvim"},
  },
  colorscheme = "kanagawa"
}
```

#### Adding vimwiki plugin

For vimwiki, I set it up so that it points to my local `~/code/tech-notes/wiki` folder, setup the task completion characters to be: ` ○◐●✓`, and modified some key bindings so that I could use `<Tab>` and `<Shift+Tab>` for indenting and unindenting tasks:

```lua
   {
      'vimwiki/vimwiki',
      event = "BufEnter *.md",
      keys = {"<leader>ww", "<leader>wt"},
      enabled = true,
      init = function () --replace 'config' with 'init'
        vim.g.vimwiki_list = {{path = '~/code/tech-notes/wiki', syntax = 'markdown', ext = '.md'}}
        vim.g.vimwiki_listsyms = " ○◐●✓"
        vim.api.nvim_set_keymap('n', '<C-]>', '<Plug>VimwikiNextLink', {silent=true})
        vim.api.nvim_set_keymap('n', '<C-[>', '<Plug>VimwikiPrevLink', {silent=true})
        vim.api.nvim_set_keymap('n', '<S-Tab>', '<Plug>VimwikiDecreaseLvlWholeItem', {silent=true})
        vim.api.nvim_set_keymap('n', '<Tab>', '<Plug>VimwikiIncreaseLvlWholeItem', {silent=true})
      end,
    },
```

As I mentioned, vimwiki has changed my note taking workflow, which has basically been unchanged for more than a decade. I usually use a simple text file with a header line for the current date, and a set of symbols (`o`, `/` and `-`) to mark a task as not started, in progress and completed. Now I have a nicer set of symbols with more fidelity and can use the shortcut `Ctrl+Space` to mark as fully complete, or with the following keybindings a bit higher up in the config file, I can use `Ctrl+/` and `Ctrl+\` to move the status forward or backwards.

```lua
vim.api.nvim_set_keymap('n', '<C-_>', ':VimwikiIncrementListItem<CR>', {noremap = true, silent = true})
vim.api.nvim_set_keymap('n', '<C-Bslash>', ':VimwikiDecrementListItem<CR>', {noremap = true, silent = true})
```

Vimwiki also has a diary mode, where you can open your entry for the current day with `<Leader>+w,<Leader>+w` which I've begun to use to start journaliing more regularly.

![Task Management with Vimwiki](/021-nvim-is-awesome/nvim_task_management.png)

## Conclusion

Neovim with the AstroNvim configuration along with a handful of configuration tweaks has become my standard development editor for the last few weeks and I've found myself being just as, if not more, productive as my VS Code workflow which I've been using for at least five years. I have yet to even start using things like macros in earnest, so I see even more upside to using Neovim in the long term.

I definitely recommend folks give the setup a shot and see if it works for them. Vim's mode based editing can take some effort to get used to, but besides some of those quirks, modern plugins make it a fantastic development experience.

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
