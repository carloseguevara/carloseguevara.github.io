---
title: "NeoVim - Modular NeoVim configuration with Lua"
subtitle: "Improve your NeoVim configuration folder"
header-img: "img/neovim/neovim-desktop.jpg"
layout: post
author: "Carlos Guevara"
tags:
  - Vim
  - NeoVim
  - Lua
---

> You could see all the codes using in this post in [this repository](https://github.com/carloseguevara/nvim_conf).

When you try to use Vim as something more than a simple text editor, you will need to add third-party packages to add
features to Vim. For this, exist [packer](https://github.com/wbthomason/packer.nvim), a package manager that is
preferred way if you are using NeoVim.

If you read the basic use guide of this package manager, the normal way to install and configure your packages is
pulling all in a single file called `plugins.lua`.

This is enough in the firsts intend to use Vim, but if you have time using Vim you finish adding more and more packages.
Eventually, you will have a plugins file with hundreds of lines and losing the control of yours package's
configurations.

For this case, exists a different approach, grouping packages configurations in different files.

# plugins.lua

One way to call the content of other files, is using the require built-in function of lua.

If you try to use the require function without change something more, you will have issues. Some configurations won't be
read, directed errors trying to compile the lua code, etc. The `plugins.lua` file in terms to have a modular use, need
to have a different init way:

```lua
local packer = require("packer")
packer.init()
packer.reset()
local use = packer.use

require('package_configuration.lua')
require('package2_configuration.lua')
...

use 'others/plugins'
use 'others/plugins2'
...
```

In this case, we won't have problems with write configurations in other files, only we need to follow the next
structure in the other files:

```lua
local packer = require 'packer'
local use = packer.use

-- Plugins
use {
    'goolord/alpha-nvim',
    requires = { 'kyazdani42/nvim-web-devicons' },
}

-- Configurations
require'alpha'.setup(require'alpha.themes.startify'.config)
```

As you can see, we have the installation and configuration of new package out of the plugins file, so we can to group
different packages by any aspect as you preferred and have a better scalability and congruence.

## We have a big problem yet

Now we have a better solution for our configuration file structure, but we need to write for each new file of configuration a new
require function with the file name in the plugins file. That don't sound great!

To avoid this, we could write a custom function that read all the files into a folder:

```lua
local RequireFolder = {}

function RequireFolder.set(path)
    print(path)
    local pfile = io.popen('ls -a ~/.config/nvim/lua/' .. path)
    local i = 0
    for filename in pfile:lines() do
        if i > 1 then require(path .. '/' .. filename:sub(1, -5)) end
        i = i + 1
    end
    pfile:close()
end

return RequireFolder
```

With this helper function, we can re-write our plugins file as:

```lua
local packer = require('packer')
packer.init()
packer.reset()
local use = packer.use

RequireFolder = require('helpers/require_folder')
RequireFolder.set('config')

use 'others/plugins'
...
```

And that is all, now we have a configuration file structure with better scalability, and we gain the control again.
Now it is time to keep adding more packages!

## Folder Structure

The final folder structure with this approach is the follow:

```bash
.
├── README.md
├── ftplugin
│   └── python.lua
├── init.lua
├── lua
│   ├── config
│   │   ├── alpha.lua
│   │   ├── autopairs.lua
│   │   ├── comment.lua
│   │   ├── format.lua
│   │   ├── git.lua
│   │   ├── lsp.lua
│   │   ├── luahop.lua
│   │   ├── lualine.lua
│   │   ├── python.lua
│   │   ├── refactoring.lua
│   │   ├── scroll.lua
│   │   ├── telescope.lua
│   │   ├── terminal.lua
│   │   ├── theme.lua
│   │   ├── tree.lua
│   │   └── treesitter.lua
│   ├── helpers
│   │   └── require_folder.lua
│   └── plugins.lua
└── plugin
    └── packer_compiled.lua
```