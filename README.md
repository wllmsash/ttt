# ttt

Transform and filter text matching a type predicate.

## Usage

`ttt` was written with JSON log streaming in mind. Some examples:

```shell
# Add a timestamp to JSON and non-JSON logs.
$ { echo 'Application started'; echo '{ "msg": "Connection detected" }'; } | ttt --json -- jq -c ". + { ts: $(date +%s) }" | ttt --complement --json -- sed "s/^/[$(date +%s)] /"
```

```shell
# Extract message field from JSON logs only.
$ { echo 'Application started'; echo '{ "msg": "Connection detected" }'; } | ttt --filter --json -- jq -cr '.msg'
```

## Installation

Install the prerequisites:

- [Python 3.10+](https://www.python.org/downloads/)

Download the script and give it execute permissions:

```shell
# Download the script and give it execute permissions.
$ curl -fLOsS https://raw.githubusercontent.com/wllmsash/ttt/main/ttt && chmod +x ttt

# Install the script for the current user; or
$ mkdir -p $HOME/.local/bin && mv ttt $HOME/.local/bin

# Install the script for all users on the system; or to somewhere else in $PATH.
$ mv ttt /usr/local/bin
```

If installing for the current user following the [XDG specifications](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)
ensure the PATH `$HOME/.local/bin` is present in the `$PATH`.

## Help

`ttt --help`:

```text
ttt - Transform and filter text matching a type predicate.

USAGE:

  $ ttt [options] [-- <command> [args]]

  Lines matching the constructed type predicate are transformed with the provided command. If a line doesn't match the
  predicate or if no command is provided it is written back to stdout.
  Each provided type option (e.g. --json) enables matching for lines of that type. Multiple type options combine to
  match any of the provided options. If no type options are provided the --all type option is automatically added,
  matching all lines.

  Input is read from stdin. Writes to stdout and stderr are line buffered.

OPTIONS:

  -h, --help          Print the help text.

  General options:

  -c, --complement    The complement (negation) of the type predicate is used instead.
  -f, --filter        Unmatched lines are filtered out.
      --strip         Matched lines have their leading and trailing whitespace (including newlines) removed before
                      being transformed or written back to stdout.
      --strip-trailing-newline
                      Matched lines have their trailing newlines removed before being transformed or written back to
                      stdout.

  Type options:

  -a, --all           Match all lines.
  -n, --none          Match no lines.
  -j, --json          Match valid JSON lines.

EXAMPLES:

  $ ttt
  $ ttt --complement
  $ ttt --all
  $ ttt --none

    Print each line untransformed.

    - No type options is equivalent to passing the --all type option. All lines are matched, but since there is no
      command each line is printed untransformed.
    - The --complement option with no other type options is equivalent to passing the --none type option, the complement
      of the default --all type option. No lines are matched so each line is printed untransformed.
    - The --all type option is equivalent to passing no type options, covered above.
    - The --none type option is equivalent to passing the --complement type option, covered above.

  $ ttt -- <command> [args]

    Run a command on each line.

    - The -- option marks the end of the option arguments and all following arguments are used as the transform command.
    - The command runs on matched lines, in this case all lines.

    This is equivalent to:

    $ xargs -d'\n' -I{} sh -c 'printf '\''{}\n'\'' | <command> [args]'
    $ parallel -j1 'printf '\''{}\n'\'' | <command> [args]'
    $ parallel -j1 -N1 --pipe '<command> [args]'

    Since all writes to stdout and stderr are line buffered this can be used as a drop in replacement for:

    $ stdbuf -oL <command> [args]
    $ unbuffer -p <command> [args]

  $ ttt --json -- jq -c ". + { ts: $(date +%s) }" | ttt --complement --json -- sed "s/^/[$(date +%s)] /"

    Add { ..., "ts": <unix_timestamp> } to each JSON line and prepend "[<unix_timestamp] " to each non-JSON line.

    The first invocation in the pipeline transforms the JSON lines.

    - The --json type option matches JSON lines.
    - The `jq -c ". + { ts: $(date +%s) }"` command uses the jq program to add a timestamp field to each JSON object.

    The second invocation in the pipeline transforms the non-JSON lines.

    - The --complement option and --json type option matches non-JSON lines.
    - The `sed "s/^/[$(date +%s)] /"` command uses the sed program to prepend the timestamp in brackets to each text
      line.

  $ ttt --filter --json -- jq -c '.ts'

    Filter out all non-JSON lines and extract a timestamp field from each JSON line.

    - The --json type option matches JSON lines.
    - The --filter option filters out unmatched lines, in this case non-JSON lines.
    - The `jq -c '.ts'` command uses the jq program to print the 'ts' timestamp field from each JSON line.

  $ ttt --strip-trailing-newline -- wc -c

    Count the number of characters in each line, not including the newline.

    - The --strip-trailing-newline option removes the trailing newline character from each line.
    - The `wc -c` command uses the wc program to print the number of characters in each line.

    This is equivalent to:

    $ ttt -- sh -c 'tr -d '\''\n'\'' | wc -c'

  $ ttt --strip -- sed 's/$/!/'

    Strip whitespace from each line and append the '!' character.

    - The --strip option removes leading and trailing whitespace, including the newline character from each line.
    - The `sed 's/$/!/'` command uses the sed program to append '!' to each line.

    This is equivalent to:

    $ { while IFS= read -r line; do printf '%s' "$line" | sed 's/^[[:blank:]]*//;s/[[:blank:]]*$//' | tr -d '\n' | sed 's/$/!/'; done; }

    To replace the trailing newline after the transformation:

    $ ttt --strip -- sh -c 'sed '\''s/$/!\n/'\'''
```
