# Configuration

`work` is configured with a JSON file `workrc` located in `$XDG_CONFIG_HOME` (if set) or in `~/.config` (otherwise).

To see where your configuration is stored, run `work config --path`:
```
$ work config --path
/home/user/.config/workrc

$ $EDITOR (work config --path)  # edit the configuration
```


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

The hours you are expected to work on each day of the week. Used, e.g., for over-/undertime calculation.

This describes a regular workweek is *not* for public holidays or vacations. For these, use `free-days`.

### Aliases

An *alias* is a shorthand for a mode, e.g., `ls` for `list`:

```
$ work ls
Fri, 09.10.: 1 record
10:00 – 12:00 | 2 h   (project)
              = 2 h
```

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

### Macros

A *macro* is a keyword that is expanded to a full invocation of `work` with mode, arguments, and even flags.
That means it must always start with a valid mode.

### Rounding precision

When using `now` for the start/stop time, the current time is rounded (down when starting, up when stopping).
`rounding_precision` sets the precision of this rounding, in minutes from `1` to `60`.

E.g., with a rounding precision of 10 and a time of 12:09, `start now` will start the run at 12:00.
