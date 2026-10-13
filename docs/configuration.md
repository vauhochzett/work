# Configuration

`work` is configured with a JSON file `workrc` located in `$XDG_CONFIG_HOME` (if set) or in `~/.config` (otherwise).

To see where your configuration is stored, run `work config --path`:
/// html | div.highlight.term
    markdown: html

<pre><code>$ <span class="t-cmd">work</span> <span class="t-param">config --path</span>
/home/user/.config/workrc

$ <span class="t-cmd">$EDITOR</span> <span class="t-op">(</span><span class="t-cmd">work</span> <span class="t-param">config --path</span><span class="t-op">)</span>  # edit the configuration
</code></pre>
///


## Default configuration

```json
{
	"expected_hours": {
		"Monday":    8.0,
		"Tuesday":   8.0,
		"Wednesday": 8.0,
		"Thursday":  8.0,
		"Friday":    8.0,
		"Saturday":  0.0,
		"Sunday":    0.0
	},
	"aliases": {
		"status": ["s"],
		"hours":  ["h"],
		"list":   ["ls"],
		"edit":   ["e"],
		"remove": ["rm"]
	},
	"macros": {
		"day": "list --include-active --with-breaks --list-empty",
		"macros": "config --see macros"
	},
	"rounding_precision": 15
}
```

## Details

### Expected hours

The hours you are expected to work on each day of a regular workweek. Used, e.g., for over-/undertime calculation.

Note: If you want to manage public holidays or vacations, use `free-days`.

### Aliases

An *alias* is a shorthand for a mode, e.g., `ls` for `list`:

/// html | div.highlight.term
    markdown: html

<pre><code>$ <span class="t-cmd">work</span> <span class="t-param">ls</span>
<span class="t-bold">Fri, 09.10.: 1 record</span>
10:00 – 12:00 | 2 h   (project)
              = 2 h
</code></pre>
///

You can also define multiple aliases:

```json
{
	"aliases": {
		"start":  ["1"],
		"stop":   ["0"],
		"status": ["s", "st"]
	}
}
```

Technically, macros (see below) can be used for this purpose as well, but aliases have two advantages.
First, they are more robust as they are validated when the config is loaded.
Second, they are shown in the output of `work --help`.

### Macros

A *macro* is a keyword that is expanded to a full invocation of `work` with mode, arguments, and even flags.
That means it must always start with a valid mode.

### Rounding precision

When using `now` for the start/stop time, the current time is rounded (down when starting, up when stopping).
`rounding_precision` sets the precision of this rounding in minutes (max: 60).

The logic is to round the minutes to the next number that is divisible by the value you provide.
To illustrate:

| Precision | Rounded times          | `start now` at 10:16 |
| --------- | ---------------------- | -------------------- |
| 15        | 1:00, 1:15, 1:30, ...  | 10:15                 |
| 10        | 1:00, 1:10, 1:20, ...  | 10:10                 |
| 7         | 1:00, 1:07, 1:14, ...  | 10:14                 |
