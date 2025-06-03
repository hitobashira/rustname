# rustname
File Renaming CLI Tool (Alpha) inspired by perl's rename command

rustname Command Document
Overview

rustname is a powerful file-collective rename tool inspired by Perl's rename command. Rename files and directories based on the specified regular expression pattern. It also features dry-run mode, interactive mode, branch-banging to prevent overwrite, and a reflink backup function before a specific file system (btrfs, xfs, bcachefs, zfs) for safety.
Overall specification

    Replaced by regular expression: Replaces the file name and directory name in the form of s/search pattern/replacement string/flag .
    Recursive processing of directories: The -r or --recursive option handles the files in the specified directory and its subdirectories recursively.
    Dry-run mode: If you specify the -n or --no-act option, you can see in advance what changes without actual renaming.
    Interactive mode: If you specify the -i or --interactive option, the user will ask for confirmation before each renaming operation.
    Prevent overwrite and granting branch numbers: The --suffix (default enabled) option automatically adds branch numbers (e.g., .1, .2) to prevent file name collisions if the renamed destination file already exists. --no-suffix allows you to disable this feature.
    Advanced output: If you specify the -v or --verbose option, you can see more detailed information such as the file name, skip reason, and the status of the backup creation.
    Colored output: The --color option allows you to control whether the output is colored (auto, always, never). The default is auto , which only colors when connected to TTY.
    reflink backup: If you specify the --reflink-backup option, create a reference link copy (like a snapshot) of the original file before renaming on the reflink-enabled file system (btrfs, xfs, bcachefs, zfs). This allows you to keep the state before the change very efficiently. Unsupported file systems such as ext4 do not perform backups.

Help (--help)

The help output of rustname is as follows (it may differ slightly depending on the version and clap display).

A simple file renaming tool inspired by Perl's 'rename' utility.

Renames files based on a regular expression substitution.

Usage: rustname [OPTIONS] [PATTERN] [PATHS]...

Arguments:
  [Pattern]
          The substitution pattern (e.g., 's/old/new/').
          This is the first argument, or can be specified with -e/--expr.

  [PATHS]
          Files or directories to process.
          If a directory, it will be traversed recursively.

The Options:
  -e, --expr <PATTERN_OPTION>
          The substitution pattern (e.g., 's/old/new/').

  -n, --no-act
          Dry run: show what would be done without actually renaming.

  -r, --recursive
          Recursive: traverse directories recursively (default for directories).
          If paths include files, it will only process those files.

  -i, --interactive
          Prompt before every rename.

  -S, --suffix
          Append a suffix to prevent overwriting existing files (e.g., .1, .2).
          By default, enabled and prevented overwrite. Use --no-suffix to overwrite.

  -v, --verbose
          Show verbose output.

      --color <color>
          [default: auto] [possible values: auto, always, never]

      --reflink-backup
          Create a reflink copy as backup before renaming (only for btrfs, xfs, bcachefs, zfs).

  -h, --help
          Print help (see a summary with '-h')

  -V, --version
          Print version

How to use (examples)

Here are some concrete use cases for rustname. The command assumes ./target/release/rustname or cargo run --release --release --
1. Basic file name replacement

Rename foo.txt in the current directory to bar.txt.
Bash

touch foo.txt
./target/release/rustname 's/foo/bar/' foo.txt
# Or if you want to find a match from the entire current directory
# ./target/release/rustname 's/foo/bar/' .

2. Check in dry-run mode

Before actually renaming, check what will be changed.
Bash

touch doc_v1.txt
./target/release/rustname 's/_v1/_final/' doc_v1.txt -n

3. Case-insensitive substitutions

Change all REPORTs and reports in the file name to document . /i Add a flag.
Bash

touch MyReport.pdf YOUR_report.docx
./target/release/rustname 's/report/document/i' MyReport.pdf YOUR_report.docx

4. Recursively handling directories

Change all .txt files in the my_docs directory and its subdirectories to .md.
Bash

mkdir -p my_docs/chapter1
touch my_docs/intro.txt my_docs/chapter1/summary.txt
./target/release/rustname 's/\.txt$/\.md/' my_docs/ -r

5. Rename while checking in interactive mode

Before each renaming operation, the user is asked to confirm.
Bash

touch image_001.jpg image_002.jpg
./target/release/rustname 's/image/photo/' . -i

You will be prompted. Rename with y and skip with n.

[Original Pass]┃
[New pass]┃
Rename this file? (y/n)

6. Granting branch number when file name collision (default operation)

If existing.txt exists, when you try to rename temp.txt to existing.txt, it will automatically have a branch number like existing.1.txt.
Bash

touch temp.txt existing.txt
./target/release/rustname 's/temp/existing/' temp.txt
# Result: temp.txt is renamed existing.1.txt

7. Disable branch numbering and allow overwriting to existing files (deprecated)

It is usually not recommended for safety.
Bash

touch temp.txt existing.txt
./target/release/rustname 's/temp/existing/' temp.txt --no-suffix
# A warning is displayed and the name is skipped
# (because there is logic to skip if the new path is already there)

Note: rustname prevents file overwriting by default. Even if you specify --no-suffix, if you try to rename an already existing file name, it will be skipped by default. This behavior is different from the overwrite behavior of the cp command, such as --force .
8. Check the detailed processing status

View in detail the files you are processing, skipped files, backup status, and more.
Bash

./target/release/rustname 's/old/new/' . -r -v

9. Create a reflink backup

Before renaming, create a reflink copy of the original file. It must be run on either the file systems of btrfs, xfs, bcachefs, or zfs.
Bash

# Run on reflink compatible file system
touch important_report.dat
./target/release/rustname 's/report/final_report/' important_report.dat --reflink-backup --verbose
# Result: important_report.dat is renamed to final_report.dat,
# Create backups like important_report.dat.YYYYYYMMDD_HHMMSS.bk

For non-compliant file systems such as ext4:
