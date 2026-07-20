# Using R in VSCode

Below is a guide to my preferred setup

# Installation

## R VSCode Extension

Install R extension [here](https://marketplace.visualstudio.com/items?itemName=REditorSupport.r)

## R terminal application

R's built-in terminal has limited features and doesn't handle multi-line statements well on some operating systems. Thus, while you can use R's built in `Rterm` console, another console program is recommended. Two common options are [radian](https://github.com/randy3k/radian) and [arf](https://github.com/eitsupi/arf). `radian` is no longer under development and `arf` is faster, though `arf` is at an earlier stage of development. I recommend using `arf` if possible.

## R packages

```{r}
install.packages("languageserver") # R language server
devtools::install_github("nx10/httpgd") # For displaying plots
```

# Configuration

## `.Rprofile`  setup

If you do not have an `.Rprofile` file, create it in the directory given by `Sys.getenv("HOME")`

```{r}
# Necessary to get the R terminal to attach properly
source(file.path(Sys.getenv(if (.Platform$OS.type == "windows") "USERPROFILE" else "HOME"), ".vscode-R", "init.R"))

# May need to include if using arf - see https://github.com/eitsupi/arf/issues/158
.First <- function() {
  if (
    Sys.getenv("TERM_PROGRAM") == "vscode" &&
      exists(".First.sys", envir = .GlobalEnv)
  ) {
    .GlobalEnv$.First.sys()
    if (exists(".First.sys", envir = .GlobalEnv, inherits = FALSE)) {
      rm(".First.sys", envir = .GlobalEnv)
    }
  }
}

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
  object_name_linter = NULL,
  commas_linter = NULL,
  pipe_consistency_linter = NULL,
  pipe_continuation_linter = NULL,
  indentation_linter = NULL,
  T_and_F_symbol_linter = NULL,
  quotes_linter = NULL
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

1. Search for `Rterm` and enter the path to your `radian` or `arf` executable.
1. Check `Always Use Active Terminal`
1. Check `Bracketed Paste`
1. Check `Plot: Use Httpgd`

## Using the terminal

If you run a line of code with `Ctrl+Enter` it will open a new `radian` or `arf` instance in your terminal. If this does not work press `Ctrl+Shift+P` and select `Create R terminal`. I prefer to have the terminal in a normal tab so I can move it around the screen, similar to the Jupyter Interactive Window. To do this, click and drag the `R Interactive` label in the top right of your console window into the text editor area.

When using this method to create a terminal plots with `httpgd` will not work unless you have setup your `~/.Rprofile` file as described above.

To speed up text rendering in the console, you may need to change the setting `Terminal: Integrated: Gpu Acceleration` to `off`. 

If text within the console is wrapping too much, you can set a wider width using something like `options(width=120)`

## Plotting

* Make sure you `Plot: Use Httpgd` is checked in Settings.
* `ggplot2` text may look too small by default. You can change the default via a command like `theme_set(theme(base_size=18))`