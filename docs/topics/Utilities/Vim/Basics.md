<!--
SPDX-FileCopyrightText: 2020 - 2023 Eli Array Minkoff

SPDX-License-Identifier: MIT
-->

# Basic how-to for Vi/Vim

> When I first wrote this, I had been using Linux systems for some time, but had found **vi/m** to be quite unintuitive. A few months later, I am using vim with various plugins as my go-to text editor.

## Switching between modes

* **command mode**: press the escape key to switch to **command mode**.
* **ex mode**: while in **command mode**, type `:`.
* **insert mode**: while in **command mode** type `i`.

## Command mode navigation

| key                | effect            |
|--------------------|-------------------|
| `h`, `left arrow`  | move cursor left  |
| `j`, `up arrow`    | move cursor up    |
| `k`, `down arrow`  | move cursor down  |
| `l`, `right arrow` | move cursor right |

## Useful ex mode commands

| command     | effect              |
|-------------|---------------------|
| `q`         | quit vi             |
| `w`         | 'write out' / save  |
| `wq`, `x`   | save and quit       |
| `q!`        | quit without saving |
| `r <file>`  | read in **file**    |
| `w <file>`  | write to **file**   |
| `w! <file>` | overwrite **file**  |

***

Sources:
* https://www.cyberciti.biz/faq/save-file-in-vi-vim-linux-apple-macos-unix-bsd/
* https://kb.iu.edu/d/adxz
