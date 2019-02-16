# git-ds

A git command extension that saves a diff of all uncommitted changes (staged, unstaged, and untracked) in the current working tree. This diff can then be shared/synced in order to pick up where you left off on a different machine, to pass the changes onto someone else for completion and commit, or any other reason you can think of.


## Installation

1. Clone or download this repository.
2. Make the `git-ds` file executable:

    `chmod +x git-ds`

3. Add `git-ds` to the system `PATH`, either by adding the repository directory to the `PATH` or by copying/linking the file from  somewhere already on the `PATH`.

After successful installation, a new command will be available in all git repositories on the system: `git ds`. The command requires it be executed in the root directory of a valid git repository.


## File rotation

Diff files are created in subdirectories named after the current branch, and with a timestamp in the filename, e.g.:

    master/20190207-211436.patch

Not only does this help with identifying the diff, but it prevents new diff files from overriding those created earlier, or in a different branch.

Keeping a separate file per generated diff provides for a certain level of historical record. This is particularly useful if, for example, the most recently generated diff occurred after a stash, and as such was empty. You may want, at a later date and on different machine, to continue work on the files as they existed prior to the stash. They could be retrieved using the next-to-last diff file.

Obviously, diff files can't just pile up forever. By default, 10 files will be kept per branch, with the oldest file being removed prior to writing out a new file. The number of files kept per branch is [configurable](#configuration).

New diff files will also *not* be created if the generated diff is identical to the last one that was saved.


## Manual execution

A diff of all uncommitted changes in the current working tree can be generated at any time using the `--run` subcommand:

    git ds --run

This will write out a file containing the generated diff, per the constraints noted in [File rotation](#file-rotation). The output location and number of files to maintain can be [configured separately](#configuration).

In order to aid in the identification of specific diffs, a label can be appended to the default filename of the generated diff file. The label can be specified using the `-l` option:

    git ds --run -l special

The above command would generate a filename that looked something like:

    master/20190207-211436-special.patch


## Configuration

Various aspects of the functionality provided by `git ds` can be customised. These settings can be configured once, via the `--config` subcommand, and will be respected by all subsequent commands.

For example, to change the maximum number of diff files maintained per branch from 10 (the default) to 20, you would use the command:

    git ds --config -n 20

Any subsequent calls to the `--run` subcommand will then respect the new value.

The available configuration options are:

Option | Default Value | Description
--- | --- | ---
`-o` | `.git/ds` | The output directory, as an absolute path or relative to the repository root directory. This is the directory to which branch subdirectories and diff files will be written.
`-n` | `10` | The maximum number of diff files to maintain per branch. Once more than this number of diff files are generated, the oldest files will be removed to make room for the new ones.

Any number of these options can be passed to `--config` at the same time.


## Automation

The generation of diff files can be automated using a periodic cronjob. This saves needing to remember to run the `--run` subcommand when necessary. The `--watch` subcommand can be used to create such a cronjob automatically:

    git ds --watch

By default, the cronjob will be scheduled to run every 5 minutes. The schedule can be configured by passing one of the following options:

Option |  Description
--- | ---
`-m` | The cronjob interval in minutes.
`-s` | The full cronjob schedule, if a more complex schedule than "every X minutes" is required.

Examples:

* Every 15 minutes:

        git ds --watch -m 15

* Every 5 minutes during work hours:

        git ds --watch -s "*/5 8-18 * * 1-5"

Tip: Use [crontab.guru](https://crontab.guru/) to validate schedule strings.

It is not valid to pass multiple scheduling options at once - only one of them will be used. Whichever is listed first will take precedence.

The `--watch` subcommand can be re-run within a repository that is already being watched to alter its schedule.

Note that this level of automation is not absolute. Even on an "every minute" schedule, changes can be missed if they are made and then the working tree is reset (e.g. stashing, changing branches, etc) in the time between cronjob intervals.

Once enabled, the cronjob can be disabled again using the `--unwatch` subcommand:

    git ds --unwatch


## Status

Details of currently saved diff files can be displayed by calling the command with no additional arguments:

    git ds

This will display:

* An indication of whether or not the repository is being [watched](#automation).
* The names of all branches for which there are saved diff files. Note: this may include branches that have since been deleted from the repository. Obsolete diff files can be removed by running the [`--clean` subcommand](#cleaning)
* The number of saved diff files in each branch, and the modification timestamp of the most recent.


## Cleaning

As branches are merged/abandoned and then deleted from the repository, saved diff files from earlier states of those branches will become obsolete. All obsolete files can be quickly and easily removed with the `--clean` subcommand:

    git ds --clean

This will display a report of the obsolete branches, details of the saved diff files that exist for them, and a confirmation prompt. If confirmed, it will delete all reported files.


## Why "ds"?

What does "ds" stand for? Take your pick. Some of my favourites include:
* diff save
* diff store
* diff share
* diff sync
* death star
* disco snail
* digital sausage
