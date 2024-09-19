# ddhdedup
Use the DDH Command Line Tool to Remove Duplicate Files

## Introduction

This clog explains how you can use the command line to remove duplicate files, which are files that contain the same content as other files. Specifically, I use the bash shell in Windows Subsystem for Linux, the `ddh` command line tool written in the Rust programming language.

## DDH

DDH (Directory Differential hTool) is a library and Command Line Interface (CLI) written in Rust to locate duplicate files.

-[https://github.com/darakian/ddh]

In WSL, I think you do something like the following to install rust tooling including cargo, which manages Rust packages.

```sh
sudo snap install rustup --classic
sudo apt install cargo
rustup update
```

Then you can use `cargo` to install DDH:

```sh
cargo install --git https://github.com/darakian/ddh ddh
```

## Use Case

I mount the Windows D: drive as /mnt/d:

```sh
sudo mkdir /mnt/d
sudo mount -t drvfs d: /mnt/d
```

I use the `ddh` command line tool to generate a JSON (`-f`) file (`-o`) that lists all (`-v`) of the files under the D:\dupes (`-d`) subdirectory:

```sh
~/.cargo/bin/ddh -d /mnt/d/dupes -v all -f json -o /tmp/dupes.json
```

I use `python3` and `batcat` to evaluate the report:

```sh 
python3 -m json.tool < /tmp/dupes.json | batcat -l json
```

Duplicates appear as follows:

```json
"file_paths": [
  "/mnt/d/dupes/second - Copy.txt",
  "/mnt/d/dupes/second.txt"
]
```

I use grep and sed to create a script that removes any duplicates.

```sh
python3 -m json.tool < /tmp/dupes.json | grep '\"/mnt/d/dupes/.*\"\,' | sed -e 's/^\W*/rm "\//' | sed -e 's/,$//' > /tmp/remdupes.sh
```

I check the script:

```
batcat -l sh /tmp/remdupes.sh
```

I run the script:

```sh
sh /tmp/remdupes.sh
```

I unmount D:

```sh
sudo umount /mnt/d
sudo rmdir /mnt/d
```

## Caveats

This relies on the DDH command line tool including its file checksum logic.

This leaves the last instance of the file as listed in the JSON file created by the DDH command line tool.

It would likely be more efficient to remove all of the files at once, or in large batches, rather than individually.

This doesn't remove empty directories.
