# pref-by-location

<!--toc:start-->

- [pref-by-location](#pref-by-location)
  - [Requirements](#requirements)
  - [Installation](#installation)
    - [Add setup function in `yazi/init.lua`.](#add-setup-function-in-yaziinitlua)
    - [Add `keymap.toml`](#add-keymaptoml)
  - [For developers](#for-developers)
  <!--toc:end-->

This is a Yazi plugin that save these preferences by location:

- [linemode](https://yazi-rs.github.io/docs/configuration/yazi#manager.linemode)
- [sort](https://yazi-rs.github.io/docs/configuration/yazi#manager.sort_by)
- [show_hidden](https://yazi-rs.github.io/docs/configuration/yazi#manager.show_hidden)

> [!IMPORTANT]
> Minimum version: yazi v25.2.7.
>
> This plugin will conflict with folder-rules. You should remove it.
> https://yazi-rs.github.io/docs/tips#folder-rules

## Requirements

- [yazi >= 25.2.7](https://github.com/sxyazi/yazi)
- Tested on Linux.

## Installation

Install the plugin:

```sh
ya pack -a boydaihungst/pref-by-location
```

### Add setup function in `yazi/init.lua`.

Prefs is optional but the setup function is required.

```lua
require("pref-by-location"):setup({
  -- Disable this plugin completely.
  -- disabled = false -- true|false (Optional)

  -- You can backup/restore this file. But don't use same file in the different OS.
  -- save_path =  -- full path to save file (Optional)
  --       - Linux/MacOS: os.getenv("HOME") .. "/.config/yazi/pref-by-location"
  --       - Windows: os.getenv("APPDATA") .. "\\yazi\\config\\pref-by-location"

  -- You don't have to set "prefs". Just use keymaps below work just fine
  prefs = { -- (Optional)
    -- location: String | Lua pattern (Required)
    --   - Support literals full path, lua pattern (string.match pattern): https://www.lua.org/pil/20.2.html
    --     And don't put ($) sign at the end of the location. %$ is ok.
    --   - If you want to use special characters (such as . * ? + [ ] ( ) ^ $ %) in "location"
    --     you need to escape them with a percent sign (%).
    --     Example: "/home/test/Hello (Lua) [world]" => { location = "/home/test/Hello %(Lua%) %[world%]", ....}

    -- sort: {} (Optional) https://yazi-rs.github.io/docs/configuration/yazi#manager.sort_by
    --   - extension: "none"|"mtime"|"btime"|"extension"|"alphabetical"|"natural"|"size"|"random", (Optional)
    --   - reverse: true|false (Optional)
    --   - dir_first: true|false (Optional)
    --   - translit: true|false (Optional)
    --   - sensitive: true|false (Optional)

    -- linemode: "none" |"size" |"btime" |"mtime" |"permissions" |"owner" (Optional) https://yazi-rs.github.io/docs/configuration/yazi#manager.linemode
    --   - Custom linemode also work. See the example below

    -- show_hidden: true|false (Optional) https://yazi-rs.github.io/docs/configuration/yazi#manager.show_hidden

    -- Some examples:
    -- Match any folder which has path start with "/mnt/remote/". Example: /mnt/remote/child/child2
    { location = "^/mnt/remote/.*", sort = { "extension", reverse = false, dir_first = true, sensitive = false} },
    -- Match any folder with name "Downloads"
    { location = ".*/Downloads", sort = { "btime", reverse = true, dir_first = true }, linemode = "btime" },
    -- Match exact folder with name "/home/test/Videos"
    { location = "/home/test/Videos", sort = { "btime", reverse = true, dir_first = true }, linemode = "btime" },

    -- show_hidden for any folder with name "secret"
    {
	    location = ".*/secret",
	    sort = { "natural", reverse = false, dir_first = true },
	    linemode = "size",
	    show_hidden = true,
    },

    -- Custom linemode also work
    {
	    location = ".*/abc",
	    linemode = "size_and_mtime",
    },
    -- DO NOT ADD location = ".*". Which currently use your yazi.toml config as fallback.
    -- That mean if none of the saved perferences is matched, then it will use your config from yazi.toml.
    -- So change linemode, show_hidden, sort_xyz in yazi.toml instead.
  },
})
```

### Add `keymap.toml`

> [!IMPORTANT]
> Always run `"plugin pref-by-location -- save"` after changed hidden, linemode, sort

Since Yazi selects the first matching key to run, `prepend_keymap` always has a higher priority than default.
Or you can use `keymap` to replace all other keys

More information about these commands and their arguments:

- [linemode](https://yazi-rs.github.io/docs/configuration/keymap#manager.linemode)
- [sort](https://yazi-rs.github.io/docs/configuration/keymap#manager.sort)
- [hidden](https://yazi-rs.github.io/docs/configuration/keymap#manager.hidden)

```toml
[manager]
  prepend_keymap = [
    # Toggle Hidden
    { on = ".", run = [ "hidden toggle", "plugin pref-by-location -- save" ], desc = "Toggle the visibility of hidden files" },

    # Linemode
    { on = [ "m", "s" ], run = [ "linemode size", "plugin pref-by-location -- save" ],        desc = "Linemode: size" },
    { on = [ "m", "p" ], run = [ "linemode permissions", "plugin pref-by-location -- save" ], desc = "Linemode: permissions" },
    { on = [ "m", "b" ], run = [ "linemode btime", "plugin pref-by-location -- save" ],       desc = "Linemode: btime" },
    { on = [ "m", "m" ], run = [ "linemode mtime", "plugin pref-by-location -- save" ],       desc = "Linemode: mtime" },
    { on = [ "m", "o" ], run = [ "linemode owner", "plugin pref-by-location -- save" ],       desc = "Linemode: owner" },
    { on = [ "m", "n" ], run = [ "linemode none", "plugin pref-by-location -- save" ],        desc = "Linemode: none" },
    # Custom size_and_mtime linemode
    # { on = [ "u", "S" ], run = [ "linemode size_and_mtime", "plugin pref-by-location -- save" ], desc = "Show Size and Modified time" },

    # Sorting
    # { on = [ ",", "d" ], run = "plugin pref-by-location -- disable",                                               desc = "Disable this plugin" },
    # This will reset any rule of this cwd then use predefined rules in setup funtion in init.lua or fallback to default settings from yazi.toml
    { on = [ ",", "R" ], run = [ "plugin pref-by-location -- reset" ],                                                 desc = "Reset preference of cwd" },
    { on = [ ",", "m" ], run = [ "sort mtime --reverse=no", "linemode mtime", "plugin pref-by-location -- save" ], desc = "Sort by modified time" },
    { on = [ ",", "M" ], run = [ "sort mtime --reverse", "linemode mtime", "plugin pref-by-location -- save" ],    desc = "Sort by modified time (reverse)" },
    { on = [ ",", "b" ], run = [ "sort btime --reverse=no", "linemode btime", "plugin pref-by-location -- save" ], desc = "Sort by birth time" },
    { on = [ ",", "B" ], run = [ "sort btime --reverse", "linemode btime", "plugin pref-by-location -- save" ],    desc = "Sort by birth time (reverse)" },
    { on = [ ",", "e" ], run = [ "sort extension --reverse=no", "plugin pref-by-location -- save" ],               desc = "Sort by extension" },
    { on = [ ",", "E" ], run = [ "sort extension --reverse", "plugin pref-by-location -- save" ],                  desc = "Sort by extension (reverse)" },
    { on = [ ",", "a" ], run = [ "sort alphabetical --reverse=no", "plugin pref-by-location -- save" ],            desc = "Sort alphabetically" },
    { on = [ ",", "A" ], run = [ "sort alphabetical --reverse", "plugin pref-by-location -- save" ],               desc = "Sort alphabetically (reverse)" },
    { on = [ ",", "n" ], run = [ "sort natural --reverse=no", "plugin pref-by-location -- save" ],                 desc = "Sort naturally" },
    { on = [ ",", "N" ], run = [ "sort natural --reverse", "plugin pref-by-location -- save" ],                    desc = "Sort naturally (reverse)" },
    # --sensitive=no or --sensitive
    # { on = [ ",", "N" ], run = [ "sort natural --reverse=no --sensitive", "plugin pref-by-location -- save" ],                    desc = "Sort naturally" },
    { on = [ ",", "s" ], run = [ "sort size --reverse=no", "linemode size", "plugin pref-by-location -- save" ],   desc = "Sort by size" },
    { on = [ ",", "S" ], run = [ "sort size --reverse", "linemode size", "plugin pref-by-location -- save" ],      desc = "Sort by size (reverse)" },
    { on = [ ",", "r" ], run = [ "sort random --reverse=no", "plugin pref-by-location -- save" ],                  desc = "Sort randomly" },
]
```

## For developers

Trigger this plugin programmatically:

```lua
-- In your plugin:
local pref_by_location = require("pref-by-location")
-- Trigger save function
ya.manager_emit("plugin", {
  pref_by_location._id,
  args = ya.quote("save", true),
})

-- Trigger reset preference of cwd
ya.manager_emit("plugin", {
  pref_by_location._id,
  args = ya.quote("reset", true),
})



```
