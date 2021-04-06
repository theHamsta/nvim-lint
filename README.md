# nvim-lint

An asynchronous linter plugin for Neovim (>= 0.5) complementary to the
built-in Language Server Protocol support.

It uses the same API to report diagnostics as the language server client
built-in to neovim would do. Any customizations you did for
`vim.lsp.diagnostic` apply for this plugin as well.


## Motivation & Goals

With [ale][1] we already got an asynchronous linter, why write yet another one?

Because [ale][1] is a full blown linter including a language server client with
its own commands and functions.


`nvim-lint` is for cases where you use the language server protocol client
built into neovim for 90% of the cases, but you want something to fill the
remaining gaps for languages where there is no good language server
implementation or where the diagnostics reporting of the language server is
inadequate and a better standalone linter exists.


## TODO

- [ ] Add parser combinators to create a parser from a pattern
- [ ] Include more linters for languages with a lack of good language servers


## Installation

- Requires [Neovim HEAD/nightly][2]
- `nvim-lint` is a plugin. Install it like any other Neovim plugin.
  - If using [vim-plug][3]: `Plug 'mfussenegger/nvim-lint'`
  - If using [packer.nvim][4]: `use 'mfussenegger/nvim-lint'`


## Usage

Configure the linters you want to run per filetype. For example:

```lua
require('lint').linters_by_ft = {
  markdown = {'vale',}
}
```

Then setup a autocmd to trigger linting. For example:

```vimL
au BufWritePost <buffer> lua require('lint').try_lint()
```

Some linters require a file to be saved to disk, others support linting `stdin`
input. For such linters you could also define a more aggressive autocmd, for
example on the `InsertLeave` or `TextChanged` events.


## Available Linters

- [Languagetool][5]
- [Vale][8]
- [ShellCheck][10]
- [Mypy][11]


## Custom Linters

You could register custom linters by adding them to the `linters` table, but
please consider contributing a linter if it is missing.


```lua
require('lint').linters.your_linter_name = {
  cmd = 'linter_cmd',
  stdin = true -- or false if it doesn't support content input via stdin. In that case the filename is automatically added to the arguments.
  args = {}, -- list of arguments. Can contain functions with zero arguments that will be evaluated once the linter is used.
  stream = nil, -- ('stdout' | 'stderr') configure the stream to which the linter outputs the linting result.
  parser = your_parse_function
}
```

`your_parse_function` can be a function which takes two arguments:

- `output`
- `bufnr`


The `output` is the output generated by the linter command.
The function must return a list of diagnostics as specified in the [language server protocol][9]:


<details>
  <summary>Diagnostic interface description</summary>

```
export interface Diagnostic {
    /**
      * The range at which the message applies.
      */
    range: Range;

    /**
      * The diagnostic's severity. Can be omitted. If omitted it is up to the
      * client to interpret diagnostics as error, warning, info or hint.
      */
    severity?: DiagnosticSeverity;

    /**
      * The diagnostic's code, which might appear in the user interface.
      */
    code?: integer | string;

    /**
      * An optional property to describe the error code.
      *
      * @since 3.16.0
      */
    codeDescription?: CodeDescription;

    /**
      * A human-readable string describing the source of this
      * diagnostic, e.g. 'typescript' or 'super lint'.
      */
    source?: string;

    /**
      * The diagnostic's message.
      */
    message: string;

    /**
      * Additional metadata about the diagnostic.
      *
      * @since 3.15.0
      */
    tags?: DiagnosticTag[];

    /**
      * An array of related diagnostic information, e.g. when symbol-names within
      * a scope collide all definitions can be marked via this property.
      */
    relatedInformation?: DiagnosticRelatedInformation[];

    /**
      * A data entry field that is preserved between a
      * `textDocument/publishDiagnostics` notification and
      * `textDocument/codeAction` request.
      *
      * @since 3.16.0
      */
    data?: unknown;
}
```
</details>



## Alternatives

- [Ale][1]
- [efm-langserver][6]
- [diagnostic-languageserver][7]


[1]: https://github.com/dense-analysis/ale
[2]: https://github.com/neovim/neovim/releases/tag/nightly
[3]: https://github.com/junegunn/vim-plug
[4]: https://github.com/wbthomason/packer.nvim
[5]: https://languagetool.org/
[6]: https://github.com/mattn/efm-langserver
[7]: https://github.com/iamcco/diagnostic-languageserver
[8]: https://github.com/errata-ai/vale
[9]: https://microsoft.github.io/language-server-protocol/specifications/specification-current/#diagnostic
[10]: https://www.shellcheck.net/
[11]: http://mypy-lang.org/