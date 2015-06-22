---
title: Neovim smart UI protocol
author: tarruda
date: 2015-06-22
template: article.jade
---

My first blog post will be about a Neovim feature I've been working on
for the last weeks. It doesn't have an official name yet but I'm calling it
"Smart UI protocol" for now. <span class="more">

Those following Neovim development probably know that one of the project's goals
is to transform Vim into an embeddable text editor engine. It's msgpack-rpc
interface and UI protocol already make this possible to a certain extent, as
shown by projects like [SolidOak][SolidOak] or [neovim-e][neovim-e]. 

The problem with the current UI protocol is that it can only be used to mirror
Neovim main screen on top of other widget toolkits, so embedders are very
limited in what they can do regarding UI customization. To understand the
problem better, let's see a diagram with a high-level illustration of how the UI
protocol currently works:


```
+---------------+         call `ui_attach` via msgpack-rpc    +---------------+
|               |             to receive UI updates           |               |
|               | <------------------------------------------ |               |
|               |                                             |               |
|               |            instructions on how to           |               |
|               |            draw the whole screen            |               |
|               | ------------------------------------------> |               |
|               |                                             |               |
|               |               send input(:vsp)              |               |
|               | <------------------------------------------ |               |
|    Neovim     |                                             |    Embedder   |
|               |   instructions on how to echo command-line  |               |
|               |      and each character as it is typed.     |               |
|               | ------------------------------------------> |               |
|               |                                             |               |
|               |               send input(<cr>)              |               |
|               | <------------------------------------------ |               |
|               |                                             |               |
|               |    instructions on how to draw a vertical   |               |
|               |       separator and redraw the buffer       |               |
|               |          in each side of the split          |               |
|               | ------------------------------------------> |               |
+---------------+                                             +---------------+
```

That is, the embedder can't do much more than follow Neovim instructions on how
to draw both the displayed text and the user interface. This effectively limits
GUIs to being nothing but glorified terminal emulators: They can put file
explorers and menus near the editor shell, but can't display windows with
different font sizes for example.

This is greatly in part because Neovim redrawing code assumes a single
monospaced font grid to represent every part of the UI, which makes perfect
sense for terminal-only programs but is very limiting in other scenarios.
Fixing this would require significant changes to the [redrawing module][screen.c],
something that can potentially introduces many bugs and further delay a stable
release.

With that said, I've been working on what I believe to be a relatively simple
solution to the problem(won't require drastic changes to [screen.c][screen.c]):
Implement a new UI protocol that allows embedders to have full control over
window layout. This new protocol is activated via a command-line switch and
replaces the old protocol when active(not possible to have both protocols active
in the same Neovim instance). Here's an overview of how Neovim behaves when this
command-line flag is passed:

- The `--embed` flag is assumed(The builtin terminal UI is disabled and
  stdin/stdout become msgpack-rpc channels)
- The `smart_ui_attach` method creates and/or attaches to a window. It returns a
  window id, which will be sent on update notifications for that specific
  window.
- A new `smart_ui_focus` method is used by embedders to tell Neovim about which
  window is currently focused.
- Each window has its own instance of the
  [screen data structures][screen-data-structures], and window redrawing
  code([win-update][win-update], [win-line][win-line]...) only affect the
  screen structures local to the window being redrawn.
- Layout-affecting commands(`:split`, `:vsplit`, `:tabnew`) are simply forwarded
  to the embedder, which must take appropriate actions to create and attach to
  new windows and display them where appropriate.
- Command-line mode won't trigger screen redraw instructions. Instead,
  msgpack-rpc objects with high-level information about what is happening on the
  command line will be sent. For example:
  - `{"command-line": ["display-input-box"]}`
  - `{"command-line": ["update-contents", "sp"]}`
  - `{"command-line": ["display-completions", "split", "spelldump", "spellgood"...]}`
- Same for insert-mode completion, instead of redraw instructions, just sent
  completion candidates and row/column/window to display the box:
  - `{"insert-completion": ["display", 1, 40, 30, "method1", "method2"]}`
  - `{"insert-completion": ["hide"]}`

In a few words: The smart UI protocol will separate the drawing of window
contents and other user interface elements such as completion boxes and window
frames. The advantages of this scheme may not be obvious, so let me list a few
here:

- Embedders have complete freedom of how windows are displayed(custom decorators,
  floating windows...)
- Frameless, single-line windows can be created, good for using Neovim as shell
  line editor for example.
- Windows can have different fonts/sizes.
- Custom widgets for displaying the command-line and insert-mode
  completion.

Clearly embedders will have more work to implement the smart UI protocol,
that's why it will be added as a new feature instead of replacing the current
version. For simple embedding scenarios and for the builtin terminal UI the
current protocol makes things much simpler.

To show the capabilities of the new UI protocol, I'm also working on a
typescript web UI that will run in any modern web browser and communicates with
Neovim via [websockify][websockify]. If you like Neovim, stay tuned because it
is about to get a facelift!


[SolidOak]: https://github.com/oakes/SolidOak
[neovim-e]: https://github.com/coolwanglu/neovim-e
[screen.c]: https://github.com/neovim/neovim/blob/master/src/nvim/screen.c
[screen-data-structures]: https://github.com/neovim/neovim/blob/master/src/nvim/globals.h#L94-L97
[win-update]: https://github.com/neovim/neovim/blob/master/src/nvim/screen.c#L623
[win-line]: https://github.com/neovim/neovim/blob/master/src/nvim/screen.c#L2054
[websockify]: https://github.com/kanaka/websockify
