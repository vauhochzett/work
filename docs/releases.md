# Release History


## 0.99: Getting a better view

Final release version: `0.99.2.post1` (2022-06-28)

### New mode `view`

- Until now, the only way to view entries was grouped by date (with `list`).
- The new mode `view` allows other views on entries.
- This version adds the `by-category` and `balance` view.

#### `view by-category`

- View entries grouped by their category.
- Use cases:
	+ Quickly view categories used in the past in a compact form.
	+ Check how much work was done for a specific project (category), e.g. in the current week.

For example:

```
$ work view by-category --since 1.2.
category            hours        %      records
-----------------------------------------------
project-A/coding    16 h 30 m    40%    17
meetings            9 h          21%    5
∅                   7 h          17%    19
project-B/coding    4 h 30 m     10%    7
project-B/slides    3 h 30 m     8%     4
help                30 m         1%     1
-----------------------------------------------
Total               41 h                76
```

Above, `∅` is used to represent entries without a category.

We could combine this with a filter (more on the new filters below) to check how much we worked for `project-B`:

```
$ work view by-category --since 1.2. --Fc "project-B*"
category            hours        %      records
-----------------------------------------------
project-B/coding    4 h 30 m     56%    7
project-B/slides    3 h 30 m     44%    4
-----------------------------------------------
Total               8 h                 11
```

#### `view balance`

- View the balance, and how it developed, over multiple weeks or longer periods.
- Until now, the balance was only tracked over the current week. This view allows arbitrary periods for balance calculation.
- Use case: Check for accumulated overtime over a longer period of time, e.g. the past year or even since you started tracking your work.

For example, assuming it is Tuesday, 2022-02-22:

```
$ work view balance --week
Day      Date      Days    Expected    Worked       Balance graph
-----------------------------------------------------------------------
Mon      21.02.    1       8 h         8 h          |        |        |
Tue      22.02.    1       8 h         5 h 45 m     |      ==|        |
Wed      23.02.    1       8 h         -            |========|        |
Thu      24.02.    1       8 h         -            |========|        |
Fri      25.02.    1       8 h         -            |========|        |
Sat      26.02.    1       -           -            |        |        |
Sun      27.02.    1       -           -            |        |        |
-----------------------------------------------------------------------
Total              7       40 h        13 h 45 m    26 h 15 m remaining
```

Alternatively, we can take a look back to previous weeks, e.g.:

```
$ work view balance --period 3.1. 20.2.
Week     Date range         Days    Expected    Worked       Balance graph
--------------------------------------------------------------------------------
w. 01    03.01. – 09.01.    7       32 h        32 h         |        |        |
w. 02    10.01. – 16.01.    7       40 h        40 h         |        |        |
w. 03    17.01. – 23.01.    7       40 h        40 h         |        |        |
w. 04    24.01. – 30.01.    7       40 h        40 h         |        |        |
w. 05    31.01. – 06.02.    7       40 h        37 h 30 m    |       =|        |
w. 06    07.02. – 13.02.    7       40 h        42 h 30 m    |        |=       |
w. 07    14.02. – 20.02.    7       40 h        45 h         |        |=       |
--------------------------------------------------------------------------------
Total                       49      272 h       277 h        5 h overtime
```

Longer periods are grouped by month:

```
$ work view balance --period 1.9.21 31.12.2021
Month    Date range             Days    Expected    Worked       Balance graph
-------------------------------------------------------------------------------------
Sep      01.09.21 – 30.09.21    30      176 h       177 h        |        |        |
Oct      01.10.21 – 31.10.21    31      168 h       168 h        |        |        |
Nov      01.11.21 – 30.11.21    30      176 h       180 h        |        |        |
Dec      01.12.21 – 31.12.21    31      144 h       100 h        |      ==|        |
-------------------------------------------------------------------------------------
Total                           122     664 h       625 h        39 h remaining
```

### Filtering for `list` (and `view`)

- Beyond selecting the date (range), records can now optionally be filtered based on both category or message.
- New flags: `--filter-category` / `--Fc` and `--filter-message` / `--Fm`
- By default, the complete category or message must match, but glob patterns (with `*`) are supported.

For example:

```
$ work ls
Tue, 22.02.: 3 records
10:00 – 12:00 | 2 h   (project-B/slides)   "Slides for customer meeting on 2022-03-01"
12:00 – 12:30 | 3 h   (project-A/coding)
14:00 – 15:00 | 45 m                       "Cleaned office"
              = 5 h 45 m

$ work ls --Fc "project*"
Tue, 22.02.: 2 records
10:00 – 12:00 | 2 h   (project-B/slides)   "Slides for customer meeting on 2022-03-01"
12:00 – 12:30 | 3 h   (project-A/coding)
              = 5 h

$ work ls --Fc "project*" --Fm "*customer*"
Tue, 22.02.: 1 record
10:00 – 12:00 | 2 h   (project-B/slides)   "Slides for customer meeting on 2022-03-01"
              = 2 h
```

### Expanded date selection

- `add`, `edit`, and `remove` now also support `--yesterday` / `-1` and `--day`
- For `list`, `view`, and `export`:
	+ New `--since` selector equivalent to `--period X today`
	+ The `--month` selector now allows (optionally) passing a DATE, to select a month by one of its dates. E.g., `--month 1.1.` selects January of the current year.

### More robust `edit`ing

- The improved handling for multi-record editing introduced in v0.97 has been finalized.
- Most noticeable will be that now multiple entries can be edited in such a way that each individual edit would be invalid, but the final state is valid again. E.g., two records (`10–12` and `12–13`) can be shifted at once (to `10–12:30` and `12:30–13`).
- Also, if an individual edit operation encounters an error, the complete edit will be rolled back safely.

### Changes

- `--day` no longer accepts ambiguous prefixes such as "s".
- `--help`:
	+ Main program now has a "teaser message".
	+ Does not mention "protocol" any more.
- If an invalid state is detected but fixed by the user, the originally requested command is now retried automatically.
- Output:
	+ `hours`: Both `--until` and `--target` / `-8` now print the projected balance at the assumed end time.
	+ When stopping or switching, the recorded run length will be printed out, e.g.: `Stopped work at 17:30 today (30 m recorded)`
	+ When an invalid state is detected, the user will no longer be offered to "abort"; instead use Ctrl-C.
	+ `config --path` now only prints the file path, enabling `subl (work config --path)`.
- CLI:
	+ `switch`: The alias `pause` will be deprecated in an upcoming version; a warning is printed if it's used.
	+ `recess`: The primary name has been changed to `free-days`. The alias `recess` will remain as an option for now.
	+ The short flag for `--version` is now `-V`.
- This version removes the migration code for the info file introduced in v0.94.

### Fixed bugs

- `status`:
	+ Note "reduced hour day" no longer printed just because expected hours are less than 8.
	+ Note "work until" now incorporates start time of active run in calculation of end time.
- `switch`: Now checks if the switch time equals the start time of the active run (meaning the run would simply cancelled and immediately restarted at the same time) and prints a warning instead.
- `edit`: Invalid changes no longer lead to deletion of the last edited record.


## 0.98: Improved selection interface and fine-grained `hours` calculation

Final release version: `0.98.1` (2021-12-18)

### New `edit` and `remove` selection interface

- A new selection interface to improve readability (inspired by `fish`'s `history` tool).
- Now allows selection of multiple indices at once.
- By entering nothing, the selection can be immediately cancelled.

For example:

```
$ work add 10 14 -c supre
$ work add 15 17:5 -m "ji"
$ work edit
Edit mode – today

[0] 10:00 – 14:00 (supre) ""
[1] 15:00 – 17:05 () "ji"

Enter nothing to cancel, or
Enter one or more indices [0..1] separated by a space, or
Enter "all" to edit all entries.

Which entries? > 
```

### New flags for `hours` calculation

- New flag `--start` to override the assumed start time.
- New flag `--pause` to include planned breaks in the calculation.
- Assumptions for the calculations are only printed once if multiple modes are selected.
- Flag `--balance` removed – the balance is now always printed.

For example (assume it is 14:17):

```
$ work start 12
Started work at 12:00 today

$ work hours --until 17
0 m on record, 2 h 17 m active run | Balance: 5 h 43 m remaining (8 h to work today)
> You will have worked 5 h at target time 17:00.

$ work cancel
Run cancelled

$ work hours --until 17
0 m on record, 0 m active run | Balance: 8 h remaining (8 h to work today)

Assuming you start now (14:15):
> You will have worked 2 h 45 m at target time 17:00.

$ work add 10 12
Added a record from 10:00 to 12:00 today

$ work hours --until 17 --target 6 --start 14 --pause 0:30
2 h on record, 0 m active run | Balance: 6 h remaining (8 h to work today)

Assuming you start at 14:00 and take 30 m of breaks:
> You will have worked 4 h 30 m at target time 17:00.
> Work until 18:30 for a 6:00 hour day.
```

### Changes

- Corrupted files will now be detected more robustly while parsing. Specifically this covers the case that a file is formatted validly, but the contents violate assumed invariants.

### Fixed bugs

- `recess`: If a vacation was added over multiple years, the days were added to both years.
- `hours`: If a day started with overtime, an exception was raised.


## 0.97: Force actions and edit better

Final release version: `0.97.4` (released 2021-08-17)

### Force `start`, `resume`, `add`

A new `--force` flag allows intentionally overriding some warnings:

- `start --force`: Start even if a run is already active.
- `resume --force`: Resume even if a new run was already started (undo `switch`).
- `add --force`: Insert entries forcefully.
	+ If an entry is *subsumed*, remove and replace it.
	+ If an entry is *overlapped*, shorten it to make space.
	+ If an entry *contains* the forced entry, split it up and shorten the splits to make space in-between.

For example:

```
$ work ls
Fri, 28.05.: 1 records
10:00 – 18:00 | 8 h  (original)
              = 8 h

$ work add 14 15 -c split --force
Existing record split up: 10:00 – 18:00 (original) ""
  ➜  10:00 – 14:00  &  15:00 – 18:00
Added a record from 14:00 to 15:00 today

$ work ls
Fri, 28.05.: 3 records
10:00 – 14:00 | 4 h  (original)
14:00 – 15:00 | 1 h  (split)
15:00 – 18:00 | 3 h  (original)
              = 8 h
```

### New `edit` handling

- When editing `"all"`, edits are only applied after all entries were iterated to allow cancelling any time and prevent some merge conflicts.
- Category and message can now be removed when editing an entry by entering `"-"` as their new value.
- Entering nothing and just pressing Enter now accepts a change, as is suggested by `[Y/n]` in the message.

### `list` output improvements

- `--include-active` now counts active run in total
- `--only-time` now merges touching entries for output
- `--with-breaks`:
	+ now shows breaks in separate lines
	+ now omits 0 minute breaks from the output.

### Changes

- `switch` now only allows one positional argument. To add a break before restarting, use the optional argument `--start H:M`.
- The `--week` selector now allows an optional argument for the week number. If none is given, the current week is assumed.
- `hours --target` now expects a string formatted as "H:M" instead of a float.
- Entries with empty category and message are now printed as `<start> – <end> () ""`.
- `recess`: Clearer error message when removing nonexistent day
- completions: Global flags no longer suggested after sub-commands
- packaging: Test modules now omitted from package

### Known bugs

- When editing `"all"` and changing an entry so that it would be merged with its *next* neighbor, an exception occurs.


## 0.96: Expected hour management

Final release version: `0.96.3`

### Expected hour management

- New module `recess`
	+ Configure holidays, vacations, and days with reduced expected hours.
	+ The expected hours for the week and day will update accordingly.

For example:

```shell
$ work recess --add-vacation 1.2. 3.2.
$ work recess --add-reduced-day 4.2. 2.0
$ work recess --list
Vacation:
  01.02.2021
  02.02.2021
  03.02.2021
Reduced hour days:
  04.02.2021 (2.0 hours)
```

- Expected hours per weekday can now be configured in the RC file.

For example:

```json
{
  "expected_hours": {
    "Monday":    8.0,
    "Tuesday":   8.0,
    "Wednesday": 4.0,
    "Thursday":  8.0,
    "Friday":    8.0,
    "Saturday":  2.0,
    "Sunday":    0.0
  }
}
```

Check your configuration with `work config --see "expected hours"`.

### Calculation of presumed work time with `hours`

- `--until` now handles differently depending on if a run is active.
- Fixes hour calculation for runs starting in the future (`--target` and `--workday`).
- Fixes presumed start time in `--until` if no run is active.

### Changes

- Output messages for `switch` are now more abstract and intuitive.
-  `work config --expected` now prints the default configuration that is used by `work config --create`.
- **Breaking**
	+ Old configuration files are now invalid, due to added (see above) and removed (see below) keys. Create a valid configuration file with `work config --create`.
	+ Autopause functionality removed, including the configuration option


## 0.95: export and CLI convenience

- New mode `export` that enables exporting entries of any date or date range as CSV.
- `start` now understands the keyword `again`, enabling easy back-to-back entries after a run was stopped.
- Time parsing now understands military time, e.g. `1400` = 14:00.
- If a corrupted record file is detected on disk, its path is now printed out.

### Fixes

- Handles Ctrl-C more cleanly.
- Fixes bug where entering nothing in `rehash` lead to a rehash.
- Fixes bug where `list --include-active` would crash if an active run starts in the future.


## 0.94: New verification algorithm

- Verification algorithm changed: Replaced MD5 hash with a much faster Adler-32 checksum.
	+ As the hash was only meant to protect against accidental edits, priorities were shifted to increase the speed.
	+ Relative speed improvement: About 20–50 % faster, more with growing size.
	+ Absolute improvement: About 1 ms for large sets of records (5 years * 200 days * 10 records).
- Empty folders for years and months in the records directory will be removed now.
- This version removes the migration code introduced in v0.93. Make sure to migrate your records before updating.

With this update, your info file needs to be updated once. Please run `work rehash` to do that.


## 0.93: Performance improvements

- Alias "pause" for `switch`
- Internal structural changes that make listing a huge number of days quicker.
- Start-up checks slimmed down to reduce start-up time.
- Adds verification functionality for traversed record directories; if an unexpected file name or type is encountered, an error is raised.

Due to a bug in previous versions, not all entries were migrated from protocol v2 to protocol v3. It is recommended to run `work ls --period 1.1.1900 31.12.2100` once (this traverses all entries in the protocol). This will fix any leftover entries automatically.


## 0.92: switch

- New `switch` function
- The `pause` function was deprecated in favor of the new `switch` function.
- The old functionality "stop at time A, restart at time B" has been preserved.
- New: only one time argument can be passed.
	+ This enables a "switch in place" functionality.
	+ When a new task is started, this command suffices to save the completed work and immediately start a new task.


## 0.9: Category and message

Final release version: `0.91`

- Entries can now have an optional category and message.
- Both can be added when stopping a run or adding an entry.
- When listing entries, these fields can be displayed.
