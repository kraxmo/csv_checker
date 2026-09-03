# CSV Checker

`csv_checker.py` validates the structure of a delimited UTF-8 text file. It compares the number of configured delimiters in the header row with each detail row and reports whether the file is suitable for processing.

The checker supports comma-separated, pipe-separated, tab-separated, and other single-character-delimited files. It uses Python's `csv` parser, so delimiters inside correctly quoted fields do not count as field separators.

## Requirements

- Python 3.14 or later
- [uv](https://docs.astral.sh/uv/) is recommended for running the project

The project has no third-party runtime dependencies.

## How It Works

1. The first non-blank record is treated as the header.
2. The configured delimiter must appear in the header. A header without the delimiter is a `BAD` file.
3. Each non-blank detail record is parsed and compared with the header's delimiter count.
4. Empty physical rows are ignored. Rows containing empty fields, such as `A||C`, are retained and validated.
5. Optionally, every detail record can also be checked against an explicit expected delimiter count.

For example, when the delimiter is `|`, these three records each contain two delimiters and are structurally valid:

```text
name|department|location
Ada|Engineering|London
Linus|Systems|Helsinki
```

Quoted delimiters are handled correctly:

```text
name|description|status
Ada|"Uses | in text"|active
```

The second record still has two field separators because the pipe inside the quoted description is part of the field value.

## Run

Run from the repository root.

### macOS And Linux

In POSIX shells such as `zsh` and `bash`, quote special delimiters with single quotes. Quoting prevents the shell from treating the pipe as a command pipeline.

```bash
uv run csv_checker.py --filename data/goodfile --delimiter '|'
```

The delimiter defaults to a comma, so a standard CSV can be checked with:

```bash
uv run csv_checker.py --filename data/input.csv
```

Use ANSI-C quoting to pass a literal tab character:

```bash
uv run csv_checker.py --filename data/input.tsv --delimiter $'\t'
```

### Windows PowerShell

PowerShell also uses single quotes for literal strings. Quote a pipe delimiter so PowerShell does not interpret it as a pipeline.

```powershell
uv run csv_checker.py --filename data\goodfile --delimiter '|'
```

Pass a tab delimiter with PowerShell's backtick escape:

```powershell
uv run csv_checker.py --filename data\input.tsv --delimiter "`t"
```

### Windows Command Prompt

In Command Prompt (`cmd.exe`), use double quotes around a pipe delimiter. Command Prompt uses the pipe character for command pipelines.

```cmd
uv run csv_checker.py --filename data\goodfile --delimiter "|"
```

Command Prompt does not provide a direct command-line escape that expands to a literal tab. Use PowerShell for tab-delimited input, or invoke the checker from Python with a tab character supplied by your calling program.

Display command-line help:

```bash
uv run csv_checker.py --help
```

If `uv` is not being used, invoke the script with your Python interpreter instead:

```bash
python csv_checker.py --filename data/input.csv --delimiter ','
```

## Statuses And Exit Codes

The checker logs a status and immediately follows it with a short explanation.

| Status | Meaning | Process exit code |
| --- | --- | --- |
| `GOOD` | The header contains the configured delimiter and all detail records match the header delimiter count. If `--delimiter_count` is set, detail records also match that count. | `0` |
| `FAIR` | `--ignoreovercount` is enabled; no detail record has fewer delimiters than the header; records with extra delimiters are accepted. | `0` |
| `BAD` | The configured delimiter is absent from the header, a detail record differs from the header delimiter count, or a detail record does not match `--delimiter_count`. | `1` |

An unreadable filename or invalid UTF-8 input also causes the program to exit with code `1`.

## Options

| Option | Description |
| --- | --- |
| `-f`, `--filename` | Path to the input file. |
| `-d`, `--delimiter` | Input field delimiter. Default: `,`. |
| `-w`, `--write_output_file` | Write a timestamped bad-record report when the file is `BAD`. |
| `-i`, `--ignoreovercount` | Classify an overcount-only file as `FAIR` rather than `BAD`. Records with fewer delimiters still make the file `BAD`. |
| `-c`, `--delimiter_count` | Expected number of delimiters in every detail record. Default: `0`, meaning no separate expected-count check. |
| `-b`, `--batchid` | Optional identifier included in log messages. |
| `-r`, `--replacement_delimiter` | For a `GOOD` file, rewrite it using this delimiter. |
| `-k`, `--keep_original` | When used with `--replacement_delimiter`, preserve the original as `<filename>.ORIGINAL`. |

## Examples

Check a pipe-delimited file on macOS or Linux:

```bash
uv run csv_checker.py -f data/goodfile -d '|'
```

On Windows, use the PowerShell or Command Prompt quoting shown in the platform-specific sections above.

Check a comma-delimited file and require exactly two delimiters (three fields) per detail record:

```bash
uv run csv_checker.py -f data/input.csv -d ',' -c 2
```

Create an error report when invalid records are found:

```bash
uv run csv_checker.py -f data/input.csv -d ',' -w
```

The report is created next to the input file with a timestamp and the `.ERROR_DELIMITER` suffix. It contains the run log, delimiter-count summary, and header plus the identified invalid records.

Allow rows with extra fields while still rejecting rows with missing fields:

```bash
uv run csv_checker.py -f data/input.csv -d ',' --ignoreovercount
```

Convert a valid tab-delimited file to pipe-delimited format. By default, the original file is removed after replacement:

```bash
uv run csv_checker.py -f data/input.tsv -d $'\t' -r '|'
```

Add `--keep_original` to retain the source file as `data/input.tsv.ORIGINAL`:

```bash
uv run csv_checker.py -f data/input.tsv -d $'\t' -r '|' --keep_original
```

## Tests

Run the complete unit test suite from the repository root:

```bash
uv run python -m unittest test_csv_checker.py
```

Without `uv`:

```bash
python -m unittest test_csv_checker.py
```

The tests cover valid and invalid delimiter counts, quoted embedded delimiters, a missing delimiter in the header, trailing blank lines, `FAIR` handling, expected delimiter counts, status explanations, and delimiter replacement.
