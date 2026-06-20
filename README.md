# Contribution 1: lyrics: Configuration option for ReST writing #2806

**Contribution Number:** 1
**Student:** Blake Hakkila
**Issue:** https://github.com/beetbox/beets/issues/2806
**Status:** Phase IV Complete

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

- [x] Test case 1: Setting `rest_directory` in the configuration and passing an argument using `-r` using relative paths, expect output to the `-r` argument path
- [x] Test case 2: Setting `rest_directory` in the configuration using a relative path, expect output to the `rest_directory` path
- [x] Test case 3: Passing an argument using `-r` using a relative path, expect output to the `-r` argument path
- [x] Test case 4: Setting `rest_directory` in the configuration and passing an argument using `-r` using absolute paths, expect output to the `-r` argument path
- [x] Test case 5: Setting `rest_directory` in the configuration using an absolute path, expect output to the `rest_directory` path
- [x] Test case 6: Passing an argument using `-r` using an absolute path, expect output to the `-r` argument path
- [x] Test case 7: Setting `rest_directory` in the configuration and passing an argument using `-r` using user home paths, expect output to the `-r` argument path
- [x] Test case 8: Setting `rest_directory` in the configuration using a user home path, expect output to the `rest_directory` path
- [x] Test case 9: Passing an argument using `-r` using a user home path, expect output to the `-r` argument path
- [x] Test case 10: No `rest_directory` configurationg or `-r` argument, expect no output

### Manual Testing

- Tested passing an output directory by the command line using `-r` setting, and verified there was output at that path
- Tested setting `rest_directory` in `config.yaml`, and verified there was output at that path
- Tested setting `rest_directory` in `config.yaml` and passing a different path to the `-r` command line arg, verified the path passed using `-r` was where output was written
- Tested not setting `rest_directory` or passing anything using `-r` and verified there was not output
- Tested using absolute, relative, and user home paths for `rest_directory` and verified they all output to the correct locations

---

## Implementation Notes

### Week 1 Progress

This week I implemented the `rest_directory` config option by changing `default=None` to `default=self.config["rest_directory"].get(Optional(str))` and adding `"rest_directory": None` to the `self.config.add()` call in `beetsplug/lyrics.py`. I realized while doing some testing that just using `default=self.config["rest_directory"].get()` caused errors later on since other data types like ints and lists can't be passed to `Path()`, so I changed it to `default=self.config["rest_directory"].get(str)`. This caused issued with the default `None` not being allowed so I ended on `default=self.config["rest_directory"].get(Optional(str))` which allows `None` or a string. I also found that unlike the command line, when a user home path was set in the config file (`~/test` for example), the output would be written to `./~/test` instead of `~/test`. This ended up being due to the path not expanding the user home, so I added a `expanduser()` call to the `Path(opts.rest_directory)` call. 

Then I wrote tests to ensure that every combination of setting or not setting the output directory through the config file or the command line worked as expected. I also added tests to check that the config file could support relative, absolute, and user home paths since I had seen failures with relative and user home paths during my implementation. Finally I added some tests to ensure that invalid data types (types other than `None` or `str`) were rejected. My tests worked by writing a config file in the pytest temp directory, parsing that config file, parsing a command line argument line, and then ensuring that files were found in the expected subdirectory of the temp directory.

After that development I updated the documentation in `docs/plugins/lyrics.rst` and the changelog in `docs/changelog.rst` to describe the new config option.

I submitted my PR and got some feedback that my tests were more extensive than desired since they replicated some tests that check for actual file output. There were also some suggestions to remove config data type tests since thats testing the `confuse` package not `beet`, to replace config file writing/reading with setting config through `self.config`, and to use `self.run_command` to run the lyric output generation instead of `cmd.func`. Based on this feedback I refactored my tests to use a mock to only track where output would occur without actually doing any output, used the recommended `self.config` and `self.run_command`, and removed the unnecessary config data type tests.

### Code Changes

- **Files modified:**
    - [beetsplug/lyrics.py](https://github.com/hakkilab/beets/blob/rest-directory-config/beetsplug/lyrics.py)
    - [test/plugins/test_lyrics.py](https://github.com/hakkilab/beets/blob/rest-directory-config/test/plugins/test_lyrics.py)
    - [docs/plugins/lyrics.rst](https://github.com/hakkilab/beets/blob/rest-directory-config/docs/plugins/lyrics.rst)
    - [docs/changelog.rst](https://github.com/hakkilab/beets/blob/rest-directory-config/docs/changelog.rst)
- **Key commits:**
    - [lyrics: add rest_directory configuration option](https://github.com/beetbox/beets/commit/478ac8cb639a9dd9a1fc9c8294004ea3db1cd9c4)
    - [Fixed tests to remove confuse testing and add mock for RestFiles](https://github.com/beetbox/beets/commit/a7aeb6db924a932ff18e90d10660b4c499688113)
- **Approach decisions:**

    I chose to use `default=self.config["rest_directory"].get()` because it reflected how other config values were handled for the lyrics plugin. I added `Optional(str)` to the `get` call so that both `None` and string values could be parsed for the configuration, where `None` would be the default empty configuration and a string would be a output directory path. I then chose to add `expanduser()` to `Path(opts.rest_directory)` so user home paths in the config file would be handled properly.

    For tests, I initially had a set of tests that tested integration between config parsing, output directory handling, and ReST file output. I initially chose to handle config by writing config files and then passing them in to be parsed using `self.config.set_file()` and `self.config.read()` as I thought setting configuration directly through assignments to `self.config` didnt properly test that config file parsing actually worked. However, based on PR feedback, I ended up changing to direct assignment through `self.config`. I had also originally written helper code to use `cmd.parser.parse_args` and `cmd.func` to run the lyric output since that is how it was implemented in the lyrics plugin, but I got PR feedback that pointed me to a nicer `self.run_command` helper already available, so I was able to remove the repeated logic. PR feedback told me that checking actual input was a bit too extensive since other tests covered this, so I ended up using a mock instead to simply record what path was passed to the class that handles output, which lets me check the functionality without actually outputting files.

---

## Pull Request

**PR Link:** https://github.com/beetbox/beets/pull/6745

**PR Description:** Adds a `rest_directory` configuration option to the lyrics plugin that specifies a directory for ReST output, equivalent to the `-r, --write-rest` command line argument

**Maintainer Feedback:**
- 06/19/2026: Maintainer feedback was that tests are a bit too extensive due to repeated test of ReST output files, config data type handling tests are unnecessary, and I should refactor to use `self.config` for setting config and `self.run_command` to run the lyrics plugin
- 06/19/2026: I added a mock to record the output path so only the path would be checked instead of doing actual ReST output and checking the files output, I removed the unecessary data type tests, and I refactored to use `self.config` and `self.run_command` as suggested

**Status:** Awaiting review

---

## Learnings & Reflections

### Technical Skills Gained

I learned how to go through the entire open source contribution model, including issue selection, forking and dev environment setup, following contribution guidelines, and submitting a PR.

### Challenges Overcome

The hardest part was getting set up to develop for the project and learning how to write tests for the project. I was able to fix these issues by Googling for the issues I ran into with environment setup and by reading through other tests to see how they were written and what infrastructure they used.

### What I'd Do Differently Next Time

Next time, I would search through the code base more extensively to see what kind of testing helpers and infrastructure are set up so I don't replicate code that later needs to be replaced, as I missed a very useful helper for running the beets project.

---

## Resources Used

- [Beets Contributing Guide](https://github.com/beetbox/beets/blob/master/CONTRIBUTING.rst)
- [Beets Docs](https://beets.readthedocs.io/en/stable/)
- [Confuse Docs](https://confuse.readthedocs.io/en/latest/)
