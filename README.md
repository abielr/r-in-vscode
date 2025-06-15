# Using R in VSCode

Below is a guide to my preferred setup

# Installation

## R VSCode Extension

Install R extension [here](https://marketplace.visualstudio.com/items?itemName=REditorSupport.r)

## radian

See [here](https://github.com/randy3k/radian)

## R packages

```{r}
install.packages("languageserver") # R language server
devtools::install_github("nx10/httpgd") # For displaying plots
```

# Configuration

## `~/.Rprofile`  setup

```{r}
# Necessary to get the R terminal to attach properly
source(file.path(Sys.getenv(if (.Platform$OS.type == "windows") "USERPROFILE" else "HOME"), ".vscode-R", "init.R"))

# Use httpgd for plots
if (interactive() && Sys.getenv("TERM_PROGRAM") == "vscode") {
  if (requireNamespace("httpgd", quietly = TRUE)) {
    options(vsc.plot = FALSE)
    options(device = function(...) {
      httpgd::hgd(silent = TRUE)
      .vsc.browser(httpgd::hgd_url(history = FALSE), viewer = "Beside")
    })
  }
}
```

## Create `~/.lintr` file

By default there seems to be too much linting going on, this disables some. For more options click [here](https://cran.r-project.org/web/packages/lintr/vignettes/lintr.html)

```
linters: linters_with_defaults(
  line_length_linter = NULL,
  open_curly_linter = NULL,
  commented_code_linter = NULL,
  trailing_whitespace_linter = NULL,
  infix_spaces_linter = NULL,
  object_name_linter = NULL
  )
```

## Create keyboard shortcuts

Add the following to your keyboard shortcuts JSON file to mimic RStudio's keyboard shortcut for running code and inserting the pipe operator

```
{
    "key": "ctrl+enter",
    "command": "r.runSelection",
    "when": "editorTextFocus && editorLangId == 'r'"
},
{
    "key": "ctrl-shift-m",
    "command": "editor.action.insertSnippet",
    "args": {
        "snippet": " %>% "
    },
    "when": "editorLangId == 'r'"
}
```

At present when you use the pipe operator VSCode is not able to auto-indent after the first line.

## Configure R extension settings

In settings (`Ctrl+,`), do the following:

1. Search for `Rterm` and enter the path to your Radian executable.
1. Check `Always Use Active Terminal`
1. Check `Bracketed Paste`
1. Check `Plot: Use Httpgd`

## Using the terminal

If you run a line of code it will open a new Radian instance in your terminal. I prefer to have the terminal in a normal tab so I can move it around the screen. To do this, in the Command Palette (`Ctrl+Shift+P`) choose `Terminal: Create New Terminal in Editor Area to the Side`

When using this method to create a terminal plots with `httpgd` will not work unless you have setup your `~/.Rprofile` file as described above.
