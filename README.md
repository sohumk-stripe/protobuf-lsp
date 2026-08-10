# protobuf-language-server

A [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) implementation for Google Protocol Buffers, written in Go.

> This project was originally created to streamline my own workflow. Some implementations may not be optimal and the features may feel incomplete, but it serves my needs well enough as it is. That said, if you have a better solution in mind, I'd be happy to switch to yours.

- **Rust version**: <https://github.com/lasorda/protobuf-lsp>
- **Repository**: <https://github.com/lasorda/protobuf-language-server>

## Features

- Document symbol (tree view with nested message / enum support)
- Go to definition (supports nested message / enum)
- Find references
- Hover (shows the definition of message / enum, including nested types)
- Document formatting / range formatting (via `clang-format`)
- Code completion (triggered by `.`)
- Jump from a C++ header to the corresponding proto definition (only global message and enum)

## VSCode / Cursor / VSCodium users

If you use VSCode or any Open VSX-compatible editor (Cursor, VSCodium, Windsurf, ...), you can install the extension directly without building the binary yourself:

- **VSCode Marketplace**: <https://marketplace.visualstudio.com/items?itemName=panzhihao.protobuf-language-server>
- **Open VSX Registry**: <https://open-vsx.org/extension/panzhihao/protobuf-language-server>

The extension launches the `protobuf-language-server` binary found in your `PATH`. To use a custom path, set `protobuf-language-server.serverPath` in VSCode settings.

See [vscode-extension/README.md](./vscode-extension/README.md) for more details.

## Installation (build from source)

Requires Go 1.19+.

```sh
# Clean the module cache (optional)
go clean -modcache

# Install to `go env GOPATH`/bin
go install github.com/lasorda/protobuf-language-server@master
```

Make sure `$(go env GOPATH)/bin` is in your `PATH`:

```sh
# Verify
protobuf-language-server --help
```

### CLI flags

| Flag       | Description                                              | Default                  |
| ---------- | -------------------------------------------------------- | ------------------------ |
| `--stdio`  | Communicate over stdio (the default LSP transport)       | `false`                  |
| `--listen` | Listen on the given TCP address (e.g. `127.0.0.1:8080`)  | empty (use stdio)        |
| `--logs`   | Log file path                                            | `logs.DefaultLogFilePath()` |

## Editor integration

### VSCode / Cursor / VSCodium

Just install the extension (see the links above).

### Neovim + [coc.nvim](https://github.com/neoclide/coc.nvim)

Add the following to `:CocConfig`:

```jsonc
"languageserver": {
    "proto": {
        "command": "protobuf-language-server",
        "filetypes": ["proto", "cpp"],
        "settings": {
            "additional-proto-dirs": []
        }
    }
}
```

### Neovim + [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)

```lua
local configs = require('lspconfig.configs')
local util = require('lspconfig.util')

configs.protobuf_language_server = {
    default_config = {
        cmd = { 'protobuf-language-server' },
        filetypes = { 'proto', 'cpp' },
        root_dir = util.root_pattern('.git'),
        single_file_support = true,
        settings = {
            ["additional-proto-dirs"] = {
                -- extra directories to search when resolving proto imports
                -- "vendor",
                -- "third_party",
            }
        },
    }
}

require('lspconfig').protobuf_language_server.setup {
    -- your custom config
}
```

### Other LSP-compatible editors

Any LSP-aware editor (Emacs lsp-mode, Sublime LSP, Helix, ...) works. Key configuration points:

- **command**: `protobuf-language-server`
- **filetypes**: `proto` (add `cpp` if you want the C++ header jump feature)
- **settings.additional-proto-dirs**: extra directories searched when resolving proto imports (relative to the workspace root or absolute)

## Settings

| Setting                 | Type       | Default | Description                                                        |
| ----------------------- | ---------- | ------- | ------------------------------------------------------------------ |
| `additional-proto-dirs` | `string[]` | `[]`    | Extra directories searched when resolving proto imports (relative or absolute) |

## Project structure

```
.
├── main.go                # Entry point, registers LSP handlers
├── components/            # LSP handler implementations (completion / hover / ...)
├── proto/                 # proto parsing, type definitions and views
│   ├── parser/
│   ├── types/
│   └── view/
├── go-lsp/                # In-tree LSP / JSON-RPC library (used by this project)
└── vscode-extension/      # VSCode extension (TypeScript)
```

## License

MIT
