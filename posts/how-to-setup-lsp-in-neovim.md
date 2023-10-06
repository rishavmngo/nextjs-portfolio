---
title: "How to setup lsp in neovim."
date: "2023-10-07"
---

In this tutorial we will learn how to setup lsp for **auto-completion** and understand role of each package.

We will also setup **auto-formating**.

Neovim support lsp, meaning it acts as a client for lsp.

Neovim have inbuilt lsp support. But neovim does't have any opinion on how to use it. Neovim left this for users.
<br/>
<br/>
<br/>
<br/>

### Understanding what is all of this

![Big picture of lsp in neovim](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t329l5nct51k8xnlo52v.png)

When you open a file in neovim a lsp client will attach to the buffer.

The client which attach dependents on the filetype and configration in lsp.

Then the client will start talk to the language server which is installed on your machine.

The lsp respond with code completiation, error diagnostics and many more features.

<br/>

<br/>

<br/>

<br/>

### Lets start neovim and attach a lsp client to buffer

We will try to start a lsp client and attach the client to a buffer.

**init.lua**

```lua
vim.lsp.start_client({
  name = 'my-server-name',
  cmd = {'lua-language-server'},
  root_dir = vim.fs.dirname(vim.fs.find({'pyproject.toml', 'setup.py'}, { upward = true })[1]),
})

```

Parameters:

-   `name` is server name given by user, must be unique
-   `cmd` to start the language server on your machine
-   `root_dir` check for pattern
    -   `upward = true` is to start searching from upward to downward

Place **setup.py**(or any type of file which is mentioned in root_dir) beside init.lua, then lsp client will recognize the pattern and attached to the buffer (init.lua in this case).

Running `:lua =vim.lsp.get_active_clients()` will give you a lua table.

If the table is empty the lsp does't attach and running `:lua =vim.lsp.get_active_clients()` will gives empty table `{}`.

If everything goes well you can see this.

![Lsp diagnostics in neovim](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yjoog09u4zc9ufzzqg2d.png)

The lsp attached and talking to language server.

You can see warnings pop up in `init.lua`.

Or you can open any lua file , just place setup.py or pyproject.toml and lsp will get attached to the buffer.

You can also get code completions just type `<C-x><C-o>`, For some reason this does't work for me so i use `<C-o><C-n><C-p>` (next and prev completions item).

Type `vim.` place the cursor after the . and in the insert mode press `<C-o><C-n><C-p>`.

If everything is alright, you will see a pop-up open.

Like this:

![Lsp code completion popup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tc28by67mv1z7ocxucak.png)

Doing all of this for each file requires a lot of effort, which is why we have many plugins available to automate these tasks.

I will provide an introduction to each plugin and its respective role in the LSP setup.

This will give you a comprehensive understanding of the function and purpose of each plugin in the LSP setup.

<br/>

<br/>

<br/>

<br/>

### neovim/nvim-lspconfig

This is a official plugins from neovim which contains configration for servers and many more functions to help you setup lsp client and start a particular language server.

Delete all of the previous code .

Install lspconfig, I used lazy.vim here you can use your fav package manager

**init.lua**

```lua
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable", -- latest stable release
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

require("lazy").setup({
-- All of the packages goes here
	"neovim/nvim-lspconfig"
}
})
```

Now import the lspconfig

**init.lua**

```lua
{...}

-- Setup language servers.
local lspconfig = require('lspconfig')
```

Now lspconfig directly give the configration of all the available lsps with default settings. see [configrations available](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md) for your programming language.

**init.lua**

```lua
local lspconfig = require('lspconfig')

-- Python
lspconfig.pyright.setup {}

-- Typescript , Javascript
lspconfig.tsserver.setup {}

-- Lua
lspconfig.lua_ls.setup {}
```

Now you can supply custom options to the each lsp setup. see [configrations](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md) for your programming language.

**init.lua**

```lua
lspconfig.rust_analyzer.setup {
  -- Server-specific settings. See `:help lspconfig-setup`
  settings = {
    ['rust-analyzer'] = {},
  },
}
```

Now we want to setup some keymaps when a lsp client attach to buffer.

Here a autocmd is created

```lua
-- Use LspAttach autocommand to only map the following keys
-- after the language server attaches to the current buffer
vim.api.nvim_create_autocmd('LspAttach', {
  group = vim.api.nvim_create_augroup('UserLspConfig', {}),
  callback = function(ev)
    -- Enable completion triggered by <c-x><c-o>
    vim.bo[ev.buf].omnifunc = 'v:lua.vim.lsp.omnifunc'

    -- Buffer local mappings.
    -- See `:help vim.lsp.*` for documentation on any of the below functions
    local opts = { buffer = ev.buf }
    vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)
    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
    vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, opts)
    vim.keymap.set('n', '<C-k>', vim.lsp.buf.signature_help, opts)
    vim.keymap.set('n', '<space>wa', vim.lsp.buf.add_workspace_folder, opts)
    vim.keymap.set('n', '<space>wr', vim.lsp.buf.remove_workspace_folder, opts)
    vim.keymap.set('n', '<space>wl', function()
      print(vim.inspect(vim.lsp.buf.list_workspace_folders()))
    end, opts)
    vim.keymap.set('n', '<space>D', vim.lsp.buf.type_definition, opts)
    vim.keymap.set('n', '<space>rn', vim.lsp.buf.rename, opts)
    vim.keymap.set({ 'n', 'v' }, '<space>ca', vim.lsp.buf.code_action, opts)
    vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)
    vim.keymap.set('n', '<space>f', function()
      vim.lsp.buf.format { async = true }
    end, opts)
  end,
})
```

Here some useful keymaps is defined like `gd` for `going to defination`.

Again if we press `<C-o><C-n><C-p>` we will get completion popup like previously.

We want to automatic triggering of a completion popup, similar to that of IDEs.

So we have to use another package called [nvim_cmp](https://github.com/hrsh7th/nvim-cmp)

Delete the autocmd

<br/>

<br/>

<br/>

<br/>

### hrsh7th/nvim-cmp

Install nvim_cmp and few **sources**.

```lua
require("lazy").setup({

   'hrsh7th/nvim-cmp', -- Autocompletion plugin,
   'hrsh7th/cmp-nvim-lsp', -- LSP source for nvim-cmp,
   'saadparwaiz1/cmp_luasnip', -- Snippets source for nvim-cmp
   'L3MON4D3/LuaSnip', -- Snippets plugin

	"neovim/nvim-lspconfig",
}
})

```

What is mean by **sources** here

-   Nvim_cmp gets the completions from many sources like
    -   `cmp-nvim-lsp` gets data from language server
    -   `luasnip` gets data from snippets and expand snippets for nvim_cmp
    -   `buffers` gets data from buffer in neovim

Here we extend the capabilities of lsp with capabilities of cmp to leverage features of cmp.

And created a lua table which contains all the lsp server which i want to setup.

```lua
local capabilities = require("cmp_nvim_lsp").default_capabilities()
local luasnip = require 'luasnip'

local servers = {
	"lua_ls"
    "tsserver"
}

for _, lsp in ipairs(servers) do
  lspconfig[lsp].setup {
    -- on_attach = my_custom_on_attach,
    capabilities = capabilities,
  }
end
```

<br/>

<br/>

<br/>

<br/>

### Setting up Nvim-cmp

Import the Nvim-cmp

```lua
local cmp = require 'cmp'

cmp.setup {

}
```

#### Snippet

Nvim-cmp doesn't "know" how to expand a snippet, that's why we need luasnip.

This callback function gets data from snippet and here cmp need to expand it.

So we use luasnip to expand the snippet.

```lua
cmp.setup {
	snippet = {
	    expand = function(args)
	      luasnip.lsp_expand(args.body)
	    end,
	  },
}
```

#### Sources

Nvim-cmp needs sources to show suggestions to us

Sources include the response from language server , buffer and many more.

```lua

cmp.setup {
	snippet = {
	    expand = function(args)
	      luasnip.lsp_expand(args.body)
	    end,
	  },
	  sources = {
	    { name = 'nvim_lsp' },
	    {name = "buffer"},
	  },
}
```

#### Define keymaps for Nvim-cmp

```lua
cmp.setup {
	snippet = {
	    expand = function(args)
	      luasnip.lsp_expand(args.body)
	    end,
	  },
	  sources = {
	    { name = 'nvim_lsp' },
	    {name = "buffer"},
	  },

	  mapping = cmp.mapping.preset.insert({
	    ['<C-u>'] = cmp.mapping.scroll_docs(-4), -- Up
	    ['<C-d>'] = cmp.mapping.scroll_docs(4), -- Down
	    ['<C-j>'] = cmp.mapping.select_next_item(),
	    ['<C-k>'] = cmp.mapping.select_prev_item(),
	    ['<C-Space>'] = cmp.mapping.complete(),
	    ['<CR>'] = cmp.mapping.confirm {
	      behavior = cmp.ConfirmBehavior.Replace,
	      select = true,
	    },
	  }),
}

```

Finally we got auto-completion while we typing.

![nvim-cmp code completion popup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5c0pc7gfasfr46qbaavc.png)

You can see `donald` which is not a keyword just present in buffer which then suggestion by cmp.

<br/>

<br/>

<br/>

<br/>

### Format on save

Now we have auto-completion. We should setup a way to format our code.

One way to format a buffer using lsp is to call `vim.lsp.buf.format()`

Calling like this `:lua vim.lsp.buf.format()` should format the buffer.

For this we have to define a autocmd which will run `vim.lsp.buf.format()` before buffer is saved.

**async** options tells how to format, if **true** the format is **non-blocking**(when vim is ideal then the formatting of buffer will happen).

I make `async = false` because i want to format the buffer before the buffer is saved no matter what.

```lua
    vim.api.nvim_create_autocmd('BufWritePre', {
      callback = function()
        vim.lsp.buf.format {
          async = false,
        }
      end,
    })
```

This will run when any buffer is save, which we don't want because if lsp_client is not attached to the buffer then it will throw error.

So we have to define the above autocommand only when a lsp is attached to buffer.

We have to create another autocommand which will run only when a lsp attached.

```lua
vim.api.nvim_create_autocmd('LspAttach', {
  group = vim.api.nvim_create_augroup('lsp-attach-format', { clear = true }),
  -- This is where we attach the autoformatting for reasonable clients
  callback = function(args)
    local client_id = args.data.client_id
    local client = vim.lsp.get_client_by_id(client_id)
    local bufnr = args.buf

    if not client.server_capabilities.documentFormattingProvider then
      return
    end


    vim.api.nvim_create_autocmd('BufWritePre', {
      group = get_augroup(client),
      buffer = bufnr,
      callback = function()
        if not format_is_enabled then
          return
        end
        vim.lsp.buf.format {
          async = false,
          filter = function(c)
            return c.id == client.id
          end,
        }
      end,
    })
  end,
})

```

Filter in lsp.buf.format receives client and must return boolean. false will stop from request to format the buffer.

For Example:

This will not format buffers which have tsserver attached.

```lua
-- Never request typescript-language-server for formatting
vim.lsp.buf.format {
  filter = function(client) return client.name ~= "tsserver" end
}
```

That's it.

[My dot files](https://github.com/rishavmngo/dot-files)

References:

-   [nvim-lua/kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)
-   [Neovim docs for lsp](https://neovim.io/doc/user/lsp.html)
