# flutter-tools.nvim

Build flutter and dart applications in neovim using the native LSP. It adds the ability to easily
launch flutter applications, debug them, as well as extending/exposing LSP functionality such as the
widget guides, an outline view of your widgets, and hot reloading.

This plugin draws inspiration from [`emacs-lsp/lsp-dart`](https://github.com/emacs-lsp/lsp-dart), [`coc-flutter`](https://github.com/iamcco/coc-flutter) and [`nvim-metals`](https://github.com/scalameta/nvim-metals).

## New to Neovim's LSP Client?

_Skip this section if you have already configured nvim lsp._

If you haven't set up nvim's lsp client before there are a few things you should know/steps to follow
before setting up this plugin.

This plugin only enhances and adds to the functionality provided by nvim. It does not by itself provide autocompletion
or configure how errors from the language server are displayed etc. This is all handled by configuring the lsp client.

This plugin handles things like starting and managing flutter application processes allowing for hot reloading, hot restarting,
selecting devices/emulators to run as well as niceties like an outline window, widget guides etc. Other core lsp functionality has to be
configured via nvim lsp.

To set up the lsp client there are a few things to do/read:

1. Read the lsp documentation this can be found in `:h lsp` or a short summary can be found [here](https://github.com/neovim/nvim-lspconfig#lsp-overview).
1. Install an autocompletion plugin such as [`nvim-compe`](https://github.com/hrsh7th/nvim-compe).
1. (Optional) Install an LSP UI plugin such as [`lspsaga`](https://github.com/glepnir/lspsaga.nvim)

## Prerequisites

- neovim 0.5+

## Notices

- The `curved` window border style has been removed in favour of the native `rounded` option added in neovim nightly. Please update your
  neovim version if you encounter errors regarding the `rounded` option as it likely means your version is out of date.

## Installation

using `vim-plug`

```vim
Plug 'nvim-lua/plenary.nvim'
Plug 'akinsho/flutter-tools.nvim'
```

or using `packer.nvim`

```lua
use {'akinsho/flutter-tools.nvim', requires = 'nvim-lua/plenary.nvim'}
```

This plugin depends on [plenary.nvim](https://github.com/nvim-lua/plenary.nvim), please make sure it is installed.

## Warning

- flutter tools does not depend on `nvim-lspconfig`. The two can co-exist but please ensure
  you do **NOT** configure `dartls` using `lspconfig`. It will be automatically set up by this
  plugin instead.

- You might encounter issues using this plugin on the `master` channel of flutter.

## Setup

### Vimscript

```vim
lua << EOF
  require("flutter-tools").setup{} -- use defaults
EOF

```

### Lua

```lua
require("flutter-tools").setup{} -- use defaults
```

# Functionality

#### Run flutter app with hot reloading

![hot reload](./.github/hot_reload.gif)

#### Start emulators or connected devices

![flutter-devices](https://user-images.githubusercontent.com/22454918/112320203-b5f31a80-8ca6-11eb-90b8-9ac934a842da.png)

#### Visualise logs

![dev log](./.github/dev_log.png)

#### Widget guides (experimental, default: disabled)

![Widget guides](./.github/outline_guide.png)

#### Outline window

![Outline window](./.github/outline.gif)

#### Closing Tags

![closing tags](./.github/closing_tags.png)

# Usage

- `FlutterRun` - Run the current project. This needs to be run from within a flutter project.
- `FlutterDevices` - Brings up a list of connected devices to select from.
- `FlutterEmulators` - Similar to devices but shows a list of emulators to choose from.
- `FlutterReload` - Reload the running project.
- `FlutterRestart` - Restart the current project.
- `FlutterQuit` - Ends a running session.
- `FlutterOutline` - Opens an outline window showing the widget tree for the given file.
- `FlutterDevTools` - Starts a Dart Dev Tools server.
- `FlutterCopyProfilerUrl` - Copies the profiler url to your system clipboard (+ register). Note that commands `FlutterRun` and
  `FlutterDevTools` must be executed first.

### Full Configuration

Please note you do _not_ need to copy and paste this whole block, this is just to show what options are available
You can add keys from the block beneath if there is any behaviour you would like to override or change.

**NOTE:** Only one of `flutter_path` and `flutter_lookup_cmd` should be set. These two keys are two ways of solving the same
problem so will conflict if both are set.

```lua

-- alternatively you can override the default configs
require("flutter-tools").setup {
  ui = {
    -- the border type to use for all floating windows, the same options/formats
    -- used for ":h nvim_open_win" e.g. "single" | "shadow" | {<table-of-eight-chars>}
    border = "rounded",
  },
  debugger = { -- integrate with nvim dap + install dart code debugger
    enabled = false,
  },
  flutter_path = "<full/path/if/needed>", -- <-- this takes priority over the lookup
  flutter_lookup_cmd = nil, -- example "dirname $(which flutter)" or "asdf where flutter"
  widget_guides = {
    enabled = false,
  },
  closing_tags = {
    highlight = "ErrorMsg", -- highlight for the closing tag
    prefix = ">", -- character to use for close tag e.g. > Widget
    enabled = true -- set to false to disable
  },
  dev_log = {
    open_cmd = "tabedit", -- command to use to open the log buffer
  },
  dev_tools = {
    autostart = false, -- autostart devtools server if not detected
    autoopen_browser = true, -- Automatically opens devtools in the browser
  },
  outline = {
    open_cmd = "30vnew", -- command to use to open the outline buffer
  },
  lsp = {
    on_attach = my_custom_on_attach,
    capabilities = my_custom_capabilities -- e.g. lsp_status capabilities
    --- OR you can specify a function to deactivate or change or control how the config is created
    capabilities = function(config)
      config.specificThingIDontWant = false
      return config
    end,
    settings = {
      showTodos = true,
      completeFunctionCalls = true,
      analysisExcludedFolders = {<path-to-flutter-sdk-packages>}
    }
  }
}
```

You can override any options available in the `lspconfig` setup, this call essentially wraps
it and adds some extra `flutter` specific handlers and utilisation options.

**NOTE:**
By default this plugin excludes analysis of the packages in the flutter SDK. If for example
you jump to the definition of `StatelessWidget`, the lsp will not try and index the 100s (maybe 1000s) of
files in that directory. If for some reason you would like this behaviour set `analysisExcludedFolders = {}`
You cannot/should not edit the files in the sdk directly so diagnostic analysis of these file is pointless.

#### Flutter binary

In order to run flutter commands you _might_ need to pass either a _path_ or a _command_ to the plugin so it can find your
installation of flutter. Most people will not need this since it will find the executable path of `flutter` if it is in your `$PATH`.

If using something like `asdf` or some other version manager or in some other custom way,
then you need to pass in a command by specifying `flutter_lookup_cmd = <my-command>`.
If you have a full path already you can pass it in using `flutter_path`.

If you are on linux and using `snap`, this plugin will automatically set the `flutter_lookup_cmd` to `flutter sdk-path` which allows finding
`snap` installations of flutter. If this doesn't work for any reason likely an old version of flutter before this command
was added, you can set your `flutter_path` to `"<INSERT-HOME-DIRECTORY>/snap/flutter/common/flutter/bin/flutter"`
which is where this is usually installed by `snap`.

### Highlights

#### Widget guides

To configure the highlight colour you can override the `FlutterWidgetGuides` highlight group.

#### Notifications

The highlights for flutter-tools notifications and popups can be changed by overriding the default highlight groups

- `FlutterNotificationNormal` - this changes the highlight of the notification content.
- `FlutterNotificationBorder` - this changes the highlight of the notification border.
- `FlutterPopupNormal` - this changes the highlight of the popup content.
- `FlutterPopupBorder` - this changes the highlight of the popup border.
- `FlutterPopupSelected` - this changes the highlight of the popup's selected line.

### Telescope Integration

![telescope picker](https://user-images.githubusercontent.com/22454918/113897929-495a3e80-97c3-11eb-959f-9574319cd93c.png)

You can list available commands in this plugin using [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim).

In order to set this up, you can explicitly load the extension.

```lua
require("telescope").load_extension("flutter")
```

Or alternatively telescope can lazy load extension but the `Telescope` command will not autocomplete lazy loaded modules.

This can be accessed using `Telescope flutter commands` or `require('telescope').extensions.flutter.commands()`

## Debugging

_Requires nvim-dap_

```lua
-- with packer
use 'mfussenegger/nvim-dap'
```

This plugin integrates with [nvim-dap](https://github.com/mfussenegger/nvim-dap) to provide debug capabilities.
Currently if `debugger` is set to `true` in the user's config **it will expect `nvim-dap` to be installed**.
If `dap` is installed the plugin will attempt to install the debugger (Dart-Code's debugger).

To use the debugger you need to run `:lua require('dap').continue()<CR>`. This will start your app. You should then be able
to use `dap` commands to begin to debug it. For more information on how to use `nvim-dap` please read the project's README
or see `:h dap`.

Also see:

- [nvim-dap-ui](https://github.com/rcarriga/nvim-dap-ui) - a plugin which provides a nice UI for `nvim-dap`.
