# ranma-alacritty-terminal

[`alacritty_terminal`](https://crates.io/crates/alacritty_terminal), the
terminal emulation library of [Alacritty](https://github.com/alacritty/alacritty),
with one change, made for [ranma](https://github.com/masshirodev/ranma), the
terminal window manager that emulates its panes with it.

The library keeps upstream's name, so a program switches by renaming the
package and nothing else:

```toml
[dependencies]
alacritty_terminal = { package = "ranma-alacritty-terminal", version = "0.1" }
```

## The change: VS16 takes two cells

VARIATION SELECTOR-16 (U+FE0F) after a character asks for its emoji
presentation, which is two cells wide (Unicode TR51). unicode-width, ratatui,
kitty and WezTerm all count `❤️` as two; upstream keeps it in the one cell the
`❤` arrived in. A program that lays out with unicode-width (every ratatui
application) then writes the rest of the line one column left of where it
means to, and its partial redraws leave fragments behind.

Here, when VS16 follows a narrow character that has a two-cell emoji
presentation (by unicode-width's tables), the cell is widened as if the
character had arrived wide: a spacer after it and the cursor past the spacer,
the line shifted in insert mode, and at the right margin the character moved
to the next line behind a leading spacer (without auto-wrap it stays one cell).
VS16 after anything else is attached as a plain zero-width mark, as upstream
does.

The code is `Term::widen_for_emoji_presentation` in `src/term/mod.rs`, with
`has_wide_emoji_presentation` and the tests in `ranma_vs16_tests`; every
added item is marked `ranma:`.

## Versions

The fork numbers its own releases; each is built on one upstream release.
The terminal still reports the upstream version to programs (the secondary
device attributes reply, `CSI > c`), from `UPSTREAM_VERSION` in
`src/term/mod.rs` rather than the package version: a program asking which
Alacritty it runs in should get the code's real answer, not the fork's
numbering. Set it to the new tag on every rebase.

| ranma-alacritty-terminal | alacritty_terminal |
| ------------------------ | ------------------ |
| 0.1.x                    | 0.26.0             |

## Keeping up with upstream

The fork is the `ranma` branch of
[masshirodev/alacritty](https://github.com/masshirodev/alacritty), started
from upstream's tag `alacritty_terminal_v0.26.0`. For a new upstream release:
rebase the branch onto the new tag, set `UPSTREAM_VERSION` to it, run
`cargo test -p ranma-alacritty-terminal`,
bump the minor version here and in the table, publish from `alacritty_terminal/`
with `cargo publish -p ranma-alacritty-terminal`, and update ranma's
`Cargo.toml`.

## License

Apache-2.0, as upstream (`LICENSE-APACHE`). This is a modified version of
alacritty_terminal; the modifications are listed above and marked `ranma:` in
the source.
