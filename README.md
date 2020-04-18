[![Build Status](https://travis-ci.com/dandavison/delta.svg?branch=master)](https://travis-ci.com/dandavison/delta)
[![Gitter](https://badges.gitter.im/dandavison-delta/community.svg)](https://gitter.im/dandavison-delta/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

## A syntax-highlighter for git and diff output

Delta brings language syntax highlighting, within-line insertion/deletion detection, and restructured diff output to git on the command line. Here's an example of `git show` output with git configured to use delta as its pager:

<img width=800px src="https://user-images.githubusercontent.com/52205/75842654-89681a00-5d96-11ea-95a8-b18e860a148a.png" alt="image" />

<br>

## Features
|                                                | delta | git | diff-so-fancy | github/gitlab |
|------------------------------------------------|-------|-----|---------------|---------------|
| language syntax highlighting                   | ✅    | ❌  | ❌            | ✅             |
| within-line insertion/deletion detection       | ✅    | ❌  | ✅            | ✅             |
| multiple insertion/deletions detected per line | ✅    | ❌  | ❌            | ✅             |
| matching of unequal numbers of changed lines   | ✅    | ❌  | ❌            | ❌             |

In addition, delta handles traditional unified diff output.

## Installation

Executables: [Linux](https://github.com/dandavison/delta/releases/download/0.0.18/delta-0.0.18-x86_64-unknown-linux-musl.tar.gz) | [MacOS](https://github.com/dandavison/delta/releases/download/0.0.18/delta-0.0.18-x86_64-apple-darwin.tar.gz) | [Windows](https://github.com/dandavison/delta/releases/download/0.0.15/delta-0.0.15-x86_64-pc-windows-msvc.zip) | [All](https://github.com/dandavison/delta/releases)

Homebrew:
```sh
brew install git-delta
```

FreeBSD pkg(8):
```sh
pkg install git-delta
```

Archlinux (AUR):
```sh
# With AUR helper `yay`:
yay -S git-delta

# Alternatively, without an AUR helper:
git clone https://aur.archlinux.org/git-delta.git
cd git-delta
makepkg -csri
```

#### Configure git to use delta

```sh
git config --global core.pager "delta --dark"  # --light for light terminal backgrounds
```

Alternatively, you can edit your `.gitconfig` directly. Delta accepts many command line options to alter colors and other details of the output. An example is
```
[core]
    pager = delta --dark --plus-color="#012800" --minus-color="#340001" --theme="base16-ocean.dark"
```

#### Configure mercurial to use delta

Edit your `.hgrc` as follow and set the options you want for delta in it:
```
[pager]
pager = delta --dark
```

#### Windows

`less.exe` installed with git doesn't work well with `delta`. A patched version of `less.exe` and instructions for installing can be found [here](https://github.com/lzybkr/less/releases/tag/fix_windows_vt).

All git commands that display diff output should now display syntax-highlighted output. For example:
  - `git diff`
  - `git show`
  - `git log -p`
  - `git stash show -p`
  - `git reflog -p`

Delta also handles unified diff output:
```
diff -u a.txt b.txt | delta
```

<br>




## Supported languages and themes
To list the supported languages and color themes, use `delta --list-languages` and `delta --list-theme-names`. To see a demo of the color themes, use `delta --list-themes`.

delta uses the same mechanisms as [bat](https://github.com/sharkdp/bat#adding-new-syntaxes--language-definitions) for locally adding custom color themes and support for new languages: please see the [bat](https://github.com/sharkdp/bat#adding-new-syntaxes--language-definitions) documentation. You will need to install bat in order to run the `bat cache --build` command.

The languages and color themes that ship with delta are those that ship with bat. So, to propose a new language or color theme for inclusion in delta, it would need to be a helpful addition to bat, in which case please open a PR against bat.


## 24 bit color

Delta looks best if your terminal application supports 24 bit colors. See https://gist.github.com/XVilka/8346728. For example, on MacOS, iTerm2 supports 24-bit colors but Terminal.app does not.

If your terminal application does not support 24-bit color, delta will still work, by automatically choosing the closest color from those available. See the `Colors` section of the help output below.

If you're using tmux, it's worth checking that 24 bit color is  working correctly. For example, run a color test script like [this  one](https://gist.githubusercontent.com/lifepillar/09a44b8cf0f9397465614e622979107f/raw/24-bit-color.sh),  or one of the others listed [here](https://gist.github.com/XVilka/8346728). If  you do not see smooth color gradients, see the discussion at  [tmux#696](https://github.com/tmux/tmux/issues/696). The short  version is you need something like this in your `~/.tmux.conf`:
```
set -ga terminal-overrides ",xterm-256color:Tc"
```
and you may then  need to quit tmux completely for it to take effect.

<br>

## Mouse scrolling

If mouse scrolling is broken, try setting your `BAT_PAGER` environment variable to (at least) `less -R` .
See [issue #58](https://github.com/dandavison/delta/issues/58) and [bat README / "Using a different pager"](https://github.com/sharkdp/bat#using-a-different-pager).


## Using Delta with Magit
[Magit](https://github.com/magit/magit) is an excellent Git client, implemented in Emacs. Delta can be used to render diffs in magit. The current instructions are:

1. Build branch `customizable-ansi-color-conversion` of Magit:
    ```sh
    git clone https://github.com/dandavison/magit.git
    cd magit
    git checkout customizable-ansi-color-conversion
    export BUILD_MAGIT_LIBGIT=
    make
    ```

2. `M-x package-install xterm-color`

3. Build branch `magit-delta` of delta:
    ```sh
    git clone https://github.com/dandavison/delta.git
    cd delta
    git checkout magit-delta
    cargo build --release
    ```
    This creates a delta executable at `target/release/delta`

4. In the delta repo, edit the shell script `magit/delta-git` so that
   instead of simply referencing "delta" it specifies the absolute
   path to your delta executable, i.e. `/PATH/TO/delta/target/release/delta`.

5. In your emacs config file add the following:
    ```emacs-lisp
    (use-package magit
      :load-path "/PATH/TO/magit/lisp")

    (use-package xterm-color
      :load-path "/PATH/TO/xterm-color")

    (setq magit-delta-git-executable "/PATH/TO/delta/magit/delta-git")
    (load-file "/PATH/TO/delta/magit/magit-delta.el")
    (magit-delta-mode +1)
    ```

You can use `M-x magit-delta-mode` to toggle the new magit behavior.


## Options
Here's the output of `delta --help`. To use these options, add them to the delta command line in your `.gitconfig` file.

```
USAGE:
    delta [FLAGS] [OPTIONS]

FLAGS:
        --dark                      Use default colors appropriate for a dark terminal background. For more control, see
                                    the other color options.
    -h, --help                      Prints help information
        --highlight-removed         Apply syntax highlighting to removed lines. The default is to apply syntax
                                    highlighting to unchanged and new lines only.
        --light                     Use default colors appropriate for a light terminal background. For more control,
                                    see the other color options.
        --list-languages            List supported languages and associated file extensions.
        --list-theme-names          List available syntax-highlighting color themes.
        --list-themes               List available syntax highlighting themes, each with an example of highlighted diff
                                    output. If diff output is supplied on standard input then this will be used for the
                                    demo. For example: `git show --color=always | delta --list-themes`.
        --show-background-colors    Show the command-line arguments (RGB hex codes) for the background colors that are
                                    in effect. The hex codes are displayed with their associated background color. This
                                    option can be combined with --light and --dark to view the background colors for
                                    those modes. It can also be used to experiment with different RGB hex codes by
                                    combining this option with --minus-color, --minus-emph-color, --plus-color, --plus-
                                    emph-color.
    -V, --version                   Prints version information

OPTIONS:
        --commit-color <commit_color>              Color for the commit section of git output. [default: yellow]
        --commit-style <commit_style>
            Formatting style for the commit section of git output. Options are: plain, box. [default: plain]

        --file-color <file_color>                  Color for the file section of git output. [default: blue]
        --file-style <file_style>
            Formatting style for the file section of git output. Options are: plain, box, underline. [default:
            underline]
        --hunk-color <hunk_color>                  Color for the hunk-marker section of git output. [default: blue]
        --hunk-style <hunk_style>
            Formatting style for the hunk-marker section of git output. Options are: plain, box. [default: box]

        --max-line-distance <max_line_distance>
            The maximum distance between two lines for them to be inferred to be homologous. Homologous line pairs are
            highlighted according to the deletion and insertion operations transforming one into the other. [default:
            0.3]
        --minus-color <minus_color>                The background color to use for removed lines.
        --minus-emph-color <minus_emph_color>      The background color to use for emphasized sections of removed lines.
        --paging <paging_mode>
            Whether to use a pager when displaying output. Options are: auto, always, and never. The default pager is
            `less`: this can be altered by setting the environment variables BAT_PAGER or PAGER (BAT_PAGER has
            priority). [default: auto]
        --plus-color <plus_color>                  The background color to use for added lines.
        --plus-emph-color <plus_emph_color>        The background color to use for emphasized sections of added lines.
        --tabs <tab_width>
            The number of spaces to replace tab characters with. Use --tabs=0 to pass tab characters through directly,
            but note that in that case delta will calculate line widths assuming tabs occupy one character's width on
            the screen: if your terminal renders tabs as more than than one character wide then delta's output will look
            incorrect. [default: 4]
        --theme <theme>
            The code syntax highlighting theme to use. Use --theme=none to disable syntax highlighting. If the theme is
            not set using this option, it will be taken from the BAT_THEME environment variable, if that contains a
            valid theme name. Use --list-themes and --compare-themes to view available themes. Note that the choice of
            theme only affects code syntax highlighting. See --commit-color, --file-color, --hunk-color to configure the
            colors of other parts of the diff output. [env: BAT_THEME=base16]
        --24-bit-color <true_color>
            Whether to emit 24-bit ("true color") RGB color codes. Options are auto, always, and never. "auto" means
            that delta will emit 24-bit color codes iff the environment variable COLORTERM has the value "truecolor" or
            "24bit". If your terminal application (the application you use to enter commands at a shell prompt) supports
            24 bit colors, then it probably already sets this environment variable, in which case you don't need to do
            anything. [default: auto]
    -w, --width <width>
            The width (in characters) of the background color highlighting. By default, the width is the current
            terminal width. Use --width=variable to apply background colors to the end of each line, without right
            padding to equal width.

Colors
------

All delta color options work the same way. There are three ways to specify a color:

1. RGB hex code

   An example of using an RGB hex code is:
   --file-color="#0e7c0e"

2. ANSI color name

   There are 8 ANSI color names:
   black, red, green, yellow, blue, magenta, cyan, white.

   In addition, all of them have a bright form:
   bright-black, bright-red, bright-green, bright-yellow, bright-blue, bright-magenta, bright-cyan, bright-white

   An example of using an ANSI color name is:
   --file-color="green"

   Unlike RGB hex codes, ANSI color names are just names: you can choose the exact color that each
   name corresponds to in the settings of your terminal application (the application you use to
   enter commands at a shell prompt). This means that if you use ANSI color names, and you change
   the color theme used by your terminal, then delta's colors will respond automatically, without
   needing to change the delta command line.

   "purple" is accepted as a synonym for "magenta". Color names and codes are case-insensitive.

3. ANSI color number

   An example of using an ANSI color number is:
   --file-color=28

   There are 256 ANSI color numbers: 0-255. The first 16 are the same as the colors described in
   the "ANSI color name" section above. See https://en.wikipedia.org/wiki/ANSI_escape_code#8-bit.
   Specifying colors like this is useful if your terminal only supports 256 colors (i.e. doesn't
   support 24-bit color).
```

<br>


## Comparisons

(`delta --light`)



<table>
  <tr>
    <td>
      delta vs. git
    </td>
    <td>
      <img width=500px style="border: 1px solid black"
           src="https://user-images.githubusercontent.com/52205/65248525-32250480-daea-11e9-9965-1a05c6a4bdf4.png"
           alt="image" />
    </td>
    <td>
      <img width=500px style="border: 1px solid black"
           src="https://user-images.githubusercontent.com/52205/65248441-14f03600-daea-11e9-88a1-d96bbb6947f8.png"
           alt="image" />
    </td>
  </tr>
  <tr>
    <td>
      delta vs. diff-so-fancy
    </td>
    <td>
      <img width=500px style="border: 1px solid black"
           src="https://user-images.githubusercontent.com/52205/65248525-32250480-daea-11e9-9965-1a05c6a4bdf4.png"
           alt="image" />
    </td>
    <td>
      <img width=500px style="border: 1px solid black"
           src="https://user-images.githubusercontent.com/52205/65248407-07d34700-daea-11e9-9a8f-6d81f4021abf.png"
           alt="image" />
    </td>
  </tr>
  <tr>
    <td>
      delta vs. github
    </td>
    <td>
      <img width=500px style="border: 1px solid black"
           src="https://user-images.githubusercontent.com/52205/65248525-32250480-daea-11e9-9965-1a05c6a4bdf4.png"
           alt="image" />
    </td>
    <td>
      <img width=500px style="border: 1px solid black"
           src="https://user-images.githubusercontent.com/52205/65248749-9a73e600-daea-11e9-9c0d-29c8f1dea08e.png"
           alt="image" />
    </td>
  </tr>
</table>





## Credit
  https://github.com/trishume/syntect<br>
  https://github.com/sharkdp/bat<br>
  https://github.com/so-fancy/diff-so-fancy
