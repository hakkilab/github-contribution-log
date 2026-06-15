# Contribution 1: lyrics: Configuration option for ReST writing #2806

**Contribution Number:** 1
**Student:** Blake Hakkila
**Issue:** https://github.com/beetbox/beets/issues/2806
**Status:** Phase II Complete

---

## Why I Chose This Issue

I chose this issue because it looked like a nice and simple issue for first time contribution. The beets also project has thorough documentation on how to setup the build environment and contribute, which seemed like great support for a first time contributor like myself. I'm very familiar with Python so the tech stack for the beets project also lined up nicely. My main goal is I hope to get my feet wet with open source contribution and be able to have my first PR merged through this issue. I think I'll learn more about how to get set up and work through the open source contribution model and see an issue all the way through the process.

---

## Understanding the Issue

### Problem Description

The beets lyrics plugin provides a command line argument `-r directory, --write-rest directory` to specify a directory for exporting lyrics in reStructuredText (ReST) format. However, this command line argument must be provided every time the lyrics plugin is used. This issue requests that a config option be added that has the same semantics as `-r directory, --write-rest directory`, exporting lyrics in ReST format to a default directory any time the lyrics plugin is run.

### Expected Behavior

There should be a lyrics plugin config option that a user can use to specify a default directory for ReST lyric export.

### Current Behavior

Currently, there is no config option available for specifying a default ReST output directory.

### Affected Components

The lyrics plugin ([code](https://github.com/beetbox/beets/blob/master/beetsplug/lyrics.py) and [docs](https://github.com/beetbox/beets/blob/master/docs/plugins/lyrics.rst)) is the affected component.

---

## Reproduction Process

### Environment Setup

beets provides instructions for getting a development environment set up in their [contribution documentation](https://github.com/beetbox/beets/blob/master/CONTRIBUTING.rst#programming).

Their instructions for install pipx didnt work on my laptop running Linux Mint 22.3, but I found instructions that worked at the [pipx website](https://pipx.pypa.io/stable/how-to/install-pipx/) which was to run the following:
```bash
sudo apt update
sudo apt install pipx
pipx ensurepath
```

The version of poetry installed by pipx by default was also too high according to the contribution docs, so I ran the following to install packages through pipx:
```bash
pipx install "poetry<2.0.0"
pipx install poethepoet
pipx ensurepath
```

After this, I cloned my fork of the beets repository and ran the following inside the beets project folder to install dependencies and run beets:
```bash
poetry install -E docs
poetry shell
```

### Steps to Reproduce

1. Create a config file with `beet config -e` and populate it as follows:
```yaml
directory: ~/music
library: ~/data/musiclibrary.db

import:
    copy: yes

plugins: lyrics

lyrics:
    auto: true
```
2. Use `beet import <path>` to import music into beets. I downloaded "What You Want" by Kevin MacLeod from https://incompetech.com/music/royalty-free/music.html to use for my testing and imported it as is.
3. Note from the [lyrics plugin docs](https://beets.readthedocs.io/en/stable/plugins/lyrics.html) that there is no config option for a default ReST output directory.
4. Use `beet lyrics`. Note that no export occurs.
5. Use `beet lyrics -r ~/rest_directory` to export lyrics as ReST. Note that export does occur.

### Reproduction Evidence

- **Commit showing reproduction:**

  [Fork commit](https://github.com/hakkilab/beets/tree/8a44a7810c4e43f26e06f233b78a5cbbd5178495)

- **Screenshots/logs:** [If applicable]
  - Lyrics plugin docs with no mention of ReST output directory configuration https://beets.readthedocs.io/en/stable/plugins/lyrics.html
  - Console output from `beet lyrics`
  ```
  lyrics: LyricsPlugin: 🔵 Lyrics already present: Kevin MacLeod - Rock harder - What You Want
  ```
  - Console output from `beet lyrics -r ~/rest_directory`
  ```
  lyrics: LyricsPlugin: 🔵 Lyrics already present: Kevin MacLeod - Rock harder - What You Want

  ReST files generated. to build, use one of:
    sphinx-build -b html  /home/hakkilab/rest_directory /home/hakkilab/rest_directory/html
    sphinx-build -b epub  /home/hakkilab/rest_directory /home/hakkilab/rest_directory/epub
    sphinx-build -b latex /home/hakkilab/rest_directory /home/hakkilab/rest_directory/latex && make -C /home/hakkilab/rest_directory/latex all-pdf
  ```
- **My findings:**

  The lack of a configuration option for a ReST output directory was reproduced.

---

## Solution Approach

### Analysis

The root cause of the issue is a missing implementation of any ReST output configuration for the lyrics plugin. This can be seen [here](https://github.com/beetbox/beets/blob/master/beetsplug/lyrics.py#L1082)

```python
cmd.parser.add_option(
    "-r",
    "--write-rest",
    dest="rest_directory",
    action="store",
    default=None,
    metavar="dir",
    help="write lyrics to given directory as ReST files",
)
```

where `default=None` means no plugin configuration is passed in as a default to use in case the command line argument is not provided.

### Proposed Solution

I propose adding a configuration option of the form

```yaml
lyrics:
    rest_directory: <directory path>
```

by modifying the configuration setup in the [lyrics plugin code](https://github.com/beetbox/beets/blob/master/beetsplug/lyrics.py) to add a `rest_directory` option.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:**

There should be a lyrics plugin config option that a user can use to specify a default directory for ReST lyric export, but currently there is none.

**Match:**

The lyrics plugin shows other instances of arguments that can be provided in both the command line as as config option, such as `-f, --force` found [here](https://github.com/beetbox/beets/blob/master/beetsplug/lyrics.py#L1091).

**Plan:**
1. Modify the `-r, --write-rest` command line arg to use `default=self.config["rest_directory"].get()` instead of `default=None` to enable a "rest_directory" config option.
2. Modify lyrics plugin `self.config.add()` call to add `"rest_directory": None` so a lack of command line arg and config leads to no export.
3. Write tests to test that not passing in the command line arg or using a config leads to no export, and using the command line arg or lyrics config leads to ReST export.
4. Update the lyrics plugin docs to explain the new config option.

**Implement:**

[Fork branch](https://github.com/hakkilab/beets/tree/rest-directory-config)

**Review:**

beets has a guide for how to submit a PR [here](https://github.com/beetbox/beets/blob/master/CONTRIBUTING.rst#how-to-submit-your-work) which includes instructions that any contribution must be accompanied by tests, documentation updates, an update to the changelog, and checks for any linting or test failures using `poe lint`, `poe format`, and `poe test`.

**Evaluate:**

I will add automated tests to check for each combination of providing or not providing the command line argument or new config option, where no export should occur when both are missing and export should occur when either is provided.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
