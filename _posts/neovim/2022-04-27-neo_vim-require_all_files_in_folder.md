---
title: "NeoVim - Modular NeoVim configuration with Lua"
subtitle: "Inprove your NeoVim configuration folde"
layout: post
author: "Carlos Guevara"
header-style: text
tags:
  - Vim
  - NeoVim
  - Lua
---

You could see all the codes using in this post in [this repository](https://github.com/carloseguevara/nvim_conf#madewithlua).

## Read all files into a folder

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

## Folder Structure

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

## Plugins File

```lua
local packer = require("packer")
packer.init()
packer.reset()
local use = packer.use

RequireFolder = require('helpers/require_folder')
RequireFolder.set("config")

use 'others/plugins'
...
```
