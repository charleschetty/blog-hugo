---
title: "A tutorial about NeoVim"
date: 2024-02-04
description: "This tutorial is for people who want to try neovim, but don't know how to start. I will guide you step by step to make your own IDE based on NeoVim."
tags: [vim]
toc: true
---



## Introduction

Once you see a post discussing the IDE, there are always some guys who brag about VIM,  while the others curse it.
Genuine knowledge comes from practice, so I decided to give it a try.
And as you can see, I successfully became a vimer. 
I don't mean that you should use VIM; what fits is the best, but if you don't try, you will never know whether it is a piece of shit or a piece of cake.

This tutorial is for people who want to try. I will guide you step by step to make your own IDE based on NeoVim.

Before we start, please **Make Sure** you know some basic commands, such as how to edit/save file and how to exit vim. If you don't, just search it on Google.

## Plugin manager 

If you are familiar with vscode, you will know what "plugin" is. Vscode provides the user with a built-in plugin manager and official website for users to 
publish, search, and download plugins. But Neovim doesn't; you need to install a plugin manager first, and search for your desired plugin on GitHub. 

Let's create a sample file: 

```cpp
#include <iostream> 

int main(){
  std::cout<<"Hello Neovim"<<std::endl;
  return 0 ;
}

```
Open it with Neovim; as you can see, there is no line number, no completion, and only highlighting.

Edit your `~/.config/nvim/init.lua`, add a single line, then open it again. You will see the line number if there is no problem. 

```lua
vim.opt["number"]=true;
```

Now that you know how to configure Neovim, you can follow [this guide](https://github.com/folke/lazy.nvim) to install `lazy.nvim`, which is the best plugin manager for Neovim.

Now install your first plugin: [the Nord theme](https://github.com/shaunsingh/nord.nvim): 

```lua 
require("lazy").setup({
	{
		"shaunsingh/nord.nvim",
		priority = 1000,
	},
})
vim.cmd.colorscheme("nord")
```

If you don't like, you can choose your desired theme by searching on GitHub. There is a [repo](https://github.com/rockerBOO/awesome-neovim) which is highly recommended


## Completion and LSP

Completion is absolutely the most important features of IDE, but here, you should install a **completion engine** first.
I choose [nvim-cmp](https://github.com/hrsh7th/nvim-cmp) because it is written in pure lua, you can use [coq.nvim](https://github.com/ms-jpq/coq_nvim) either.

### nvim-cmp
Open the website, then you will see something like :
```vim
call plug#begin(s:plug_dir)
Plug 'neovim/nvim-lspconfig'
Plug 'hrsh7th/cmp-nvim-lsp'
Plug 'hrsh7th/cmp-buffer'
Plug 'hrsh7th/cmp-path'
Plug 'hrsh7th/cmp-cmdline'
Plug 'hrsh7th/nvim-cmp'

" For vsnip users.
Plug 'hrsh7th/cmp-vsnip'
Plug 'hrsh7th/vim-vsnip'

" For luasnip users.
" Plug 'L3MON4D3/LuaSnip'
" Plug 'saadparwaiz1/cmp_luasnip'
```
Maybe you have a question: How should I install it? If you have no idea, you must not have read `lazy.nvim`'s document carefully.
What we need is `nvim-cmp`, but it seems that they install several plugins such as `nvim-lspconfig`, `cmp-nvim-lsp` .....
Undoubtedly, that are **"dependencies"**.

Your config may like this :
```lua
require("lazy").setup({
	{
		"hrsh7th/nvim-cmp",
		config = function()
            -- todo
        end,
		dependencies = {
            {'neovim/nvim-lspconfig'},
			{ "hrsh7th/cmp-nvim-lsp" },
			{"L3MON4D3/LuaSnip",},
			{ "saadparwaiz1/cmp_luasnip" },
            -- I like LuaSnip, you can choose what you want.
			{ "hrsh7th/cmp-nvim-lsp" },
			{ "hrsh7th/cmp-path" },
			{ "hrsh7th/cmp-buffer" },
            {'hrsh7th/cmp-cmdline'},
		},
	},
})
```
But hey, do you really want to insert all config in `init.lua`? I dont't, so I choose another way.

```lua
config = require("user.cmp"),
```
Your file directory should look like : 
```
├── init.lua
├── lazy-lock.json
└── lua
    └── user
        └── cmp.lua
```
You can store your config for `nvim-cmp` in `cmp.lua`, and here is a basic config. You can easily understand it if you have read the demo carefully. 

```lua
return function()
	local cmp = require("cmp")

	cmp.setup({
		snippet = {
			expand = function(args)
				require("luasnip").lsp_expand(args.body) -- For `luasnip` users.
			end,
		},
		mapping = cmp.mapping.preset.insert({
      -- see https://github.com/hrsh7th/nvim-cmp/blob/main/doc/cmp.txt
			["<C-k>"] = cmp.mapping.select_prev_item(),
			["<C-j>"] = cmp.mapping.select_next_item(),

			["<C-e>"] = cmp.mapping.abort(),
			["<CR>"] = cmp.mapping.confirm({ select = true }), 
      -- see Example mappings in the wiki
			["<Tab>"] = cmp.mapping(function(fallback)
				if cmp.visible() then
					cmp.select_next_item()
				elseif luasnip.expand_or_jumpable() then
					luasnip.expand_or_jump()
				elseif has_words_before() then
					cmp.complete()
				else
					fallback()
				end
			end, { "i", "s" }),
		}),
		sources = cmp.config.sources({
			{ name = "nvim_lsp" },
			{ name = "luasnip" }, -- For luasnip users.
		}),
	})

	local capabilities = require("cmp_nvim_lsp").default_capabilities()
  -- see https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#clangd
	require("lspconfig")["clangd"].setup({
		capabilities = capabilities,
	})

    -- you can disable the following line, and see what happen
    vim.opt["pumheight"]=10 -- pop up menu height
end

```
Maybe you still have some questions:
- What is clangd and what is "YOUR_LSP_SERVER"? 
    - If you don't know, please read [this](https://microsoft.github.io/language-server-protocol/)
- What about `<C-e>`, `<CR>`?
    - `<C-e>` means **Ctrl** + **e**, `<CR>` means **Enter**. You can read the [official document](https://neovim.io/doc/user/map.html) or simply type the command `:help key-notation` in neovim.

Make sure `clangd` is successfully installed (**[ccls](https://github.com/MaskRay/ccls)** is ok also), then reopen the sample file, the completion will work.

However, the completion's UI is ugly, I think. Limited by article length, I can't write too much in this section; you can refer to my config to make it look better:
- [handlersmason.lua](https://github.com/charleschetty/dotfile/blob/main/config/nvim/lua/user/cmp/handlersmason.lua) 
- [cmp.lua](https://github.com/charleschetty/dotfile/blob/main/config/nvim/lua/user/cmp/cmp.lua)

### Practice: install and configure following plugins
There are some plugins I recommend, and you can install and configure them by yourself or simply copy [my config](https://github.com/charleschetty/dotfile/tree/main/config/nvim).
- [lsp_signature.nvim](https://github.com/ray-x/lsp_signature.nvim): Show function signature when you type
- [lspsaga.nvim](https://github.com/nvimdev/lspsaga.nvim): Improve neovim lsp experience 
- [guard.nvim](https://github.com/nvimdev/guard.nvim): Async formatting and linting utility for neovim
- [nvim-autopairs](https://github.com/windwp/nvim-autopairs): A super powerful autopair plugin for Neovim that supports multiple characters.
- [Comment.nvim](https://github.com/numToStr/Comment.nvim): Smart and powerful comment plugin for neovim.
- [trouble.nvim](https://github.com/folke/trouble.nvim):  A pretty diagnostics, references, telescope results, quickfix and location list to help you solve all the trouble your code is causing. 

You may have trouble dealing with the shortcuts, so I give you my keymap. 
If you have any questions, please read the document about [nvim_set_keymap](https://neovim.io/doc/user/api.html#nvim_set_keymap()).
```lua 
	local keymap = vim.api.nvim_set_keymap
	local opts = { noremap = true, silent = true }
	keymap("", "<Space>", "<Nop>", opts)
    -- https://neovim.io/doc/user/map.html#%3CLeader%3E
	vim.g.mapleader = " "
	vim.g.maplocalleader = " "
	keymap("n", "gh", "<cmd>Lspsaga lsp_finder<CR>", opts)
	keymap("n", "<leader>lf", "<cmd>Lspsaga finder def+ref<cr>", opts)
	keymap("n", "K", "<cmd>Lspsaga hover_doc<CR>", opts)
	keymap("n", "gp", "<cmd>Lspsaga peek_definition<CR>", opts)
	keymap("n", "gd", "<cmd>Lspsaga goto_definition<CR>", opts)
	keymap("n", "<leader>lr", "<cmd>Lspsaga rename<CR>", opts)
	keymap("n", "<leader>la", "<cmd>Lspsaga code_action<CR>", opts)
	keymap("n", "<leader>lj", "<cmd>Lspsaga diagnostic_jump_next<CR>", opts)
	keymap("n", "<leader>lk", "<cmd>Lspsaga diagnostic_jump_prev<CR>", opts)
	keymap("n", "<leader>o", "<cmd>Lspsaga outline<CR>", opts)
	keymap("n", "<C-\\>", "<cmd>Lspsaga term_toggle<CR>", opts)
	keymap("t", "<C-\\>", "<cmd>Lspsaga term_toggle<CR>", opts)

    -- for comment.nvim
	keymap("n", "<leader>/", "<cmd>lua require(\"Comment.api\").toggle.linewise.current()<CR>", opts)

    -- for trouble.nvim
    keymap('n', '<leader>t', ':TroubleToggle<cr>', opts)
```


## Edit multiple files

Now it seems ok to edit one single file using Neovim, but a real project may consist of several files, so how should we do that?
* We need a "file browser" to choose files.
* We need a top bar to show names and status of opened files , as other IDEs do.

### Native method
Before we start, I will show you how to edit multiple files without any plugins.

Create another sample file and open sample files using command:
```bash
nvim sample1.cpp sample2.cpp
```
Then use commands `:buffers` in Neovim, you will see something like:
```vim
:buffers
  1 %a   "sample1.cpp"                  line 1
  2      "sample2.cpp"                  line 0
Press ENTER or type command to continue
```
If you want to edit `sample2.cpp`, just type the command: `:buffer 2`, and you will see `sample2.cpp` in your editor.

How to open another file? For example, the directory is like: 
```
.
├── 1.md
├── 1.norg
├── sample1.cpp
└── sample2.cpp
```
If you want to edit `1.md`, you can use this command: `:edit 1.md`, then you can see your desired file.

You can use `:split` to split the tab horizontally and `:vsplit` split it vertically. Then use `<C-w>h`,`<C-w>j`,`<C-w>k`,`<C-w>l` to navigate the windows.
Use `:q` to quit the current window. For more details, please see the [official document](https://neovim.io/doc/user/windows.html). There is also a comprehensive [guide](https://linuxhandbook.com/split-vim-workspace/).


### With plugins


Install [bufferline.nvim](https://github.com/akinsho/bufferline.nvim), and [nvim-tree](https://github.com/nvim-tree/nvim-tree.lua), and you can see a top bar, 
and a file browser if you type the command `:NvimTreeOpen` or `:NvimTreeToggle`(Maybe it is wise to set keymaps)
```lua
require("lazy").setup({
	{
		"akinsho/bufferline.nvim",
		dependencies = { "nvim-tree/nvim-web-devicons" },
		config = function()
			vim.opt.termguicolors = true
			require("bufferline").setup({})
            -- configure it by yourself 
		end,
	},
	{
		"nvim-tree/nvim-tree.lua",
		dependencies = {"nvim-tree/nvim-web-devicons"},
		config = function()
			require("nvim-tree").setup({})
            -- configure it by yourself 
		end,
	},
})

-- my keymap
keymap("n", "<leader>e", ":NvimTreeToggle<cr>", opts)
```
Looks like having these two plugins is better than none. But that is not enough.

Install [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim). Now you can use the shortcuts to find file/buffer, grep in a very convenient way.
```lua
require("lazy").setup({
	{
		"nvim-telescope/telescope.nvim",
		dependencies = { "nvim-lua/plenary.nvim" },
	},
})
-- my keymap
keymap("n", "<leader>f",
  "<cmd>lua require'telescope.builtin'.find_files(require('telescope.themes').get_dropdown({ previewer = false }))<cr>",
  opts)
keymap("n", "<c-t>", "<cmd>Telescope live_grep<cr>", opts)
keymap("n", "<leader>b",
      "<cmd>lua require('telescope.builtin').buffers(require('telescope.themes').get_dropdown{previewer = false})<cr>",
  opts)
```

There is also another plugin I like: [nvim-bufdel](https://github.com/ojroques/nvim-bufdel), but if you don't care about the way of closing buffers, never mind.

I didn't show specific configuration of each plugin, because I think everyone has their own customization. And here is [my config](https://github.com/charleschetty/dotfile/tree/main/config/nvim)

## Other plugins and configuration 

There are still several plugins can enhance your experience, if you are interested, you can see their repo. And there are also a number of internal variables that vim has,
you can see [my conifg](https://github.com/charleschetty/dotfile/blob/main/config/nvim/lua/user/options.lua) and the [official document](https://neovim.io/doc/user/quickref.html#option-list).
- [wilder.nvim](https://github.com/gelguy/wilder.nvim): A more adventurous wildmenu
- [nvim-ufo](https://github.com/kevinhwang91/nvim-ufo): Not UFO in the sky, but an ultra fold in Neovim. (It is better to use it with [statuscol.nvim](https://github.com/luukvbaal/statuscol.nvim))
- [flash.nvim](https://github.com/folke/flash.nvim):Navigate your code with search labels, enhanced character motions and Treesitter integration 
    - The best plugin for searching
- [nvim-spider](https://github.com/chrisgrieser/nvim-spider): Use the `w`, `e`, `b` motions like a spider. Move by subwords and skip insignificant punctuation. 
- [vim-matchup](https://github.com/andymass/vim-matchup): vim match-up: even better `%` navigate and highlight matching words modern matchit and matchparen. Supports both vim and neovim + tree-sitter. 

The following plugins are for better look.

- [aplha-nvim](https://github.com/goolord/alpha-nvim): A fast and fully programmable greeter for neovim.
- [Indent Blankline](https://github.com/lukas-reineke/indent-blankline.nvim): This plugin adds indentation guides to Neovim
- [lualine.nvim](https://github.com/nvim-lualine/lualine.nvim): A blazing fast and easy to configure Neovim statusline written in Lua.
- [which-key.nvim](https://github.com/folke/which-key.nvim): WhichKey is a lua plugin for Neovim 0.5 that displays a popup with possible key bindings of the command you started typing.
    - If you are not familiar with your keymaps, maybe this plugin is useful
- [illuminate.vim](https://github.com/RRethy/vim-illuminate): Neovim plugin for automatically highlighting other uses of the word under the cursor using either LSP, Tree-sitter, or regex matching.
- [nvim-hlslens](https://github.com/kevinhwang91/nvim-hlslens): nvim-hlslens helps you better glance at matched information, seamlessly jump between matched instances.

### FAQ 

- Debug?  
    - I use [gdb-dashboard](https://github.com/cyrus-and/gdb-dashboard), Clion, VSc to debug, if you want to debug your program in neovim, you can use [nvim-dap](https://github.com/mfussenegger/nvim-dap).
- How to (compile and )run my program in neovim? 
    - I use a specific terminal to run and test my program, if you'd like to do it with neovim, you can use [code_runner.nvim](https://github.com/CRAG666/code_runner.nvim)

