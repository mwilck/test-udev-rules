# test program for SCSI udev rules

This is a test program for the udev rules proposed for 
[systemd](https://github.com/systemd/systemd/pull/7594) and 
[sg3_utils](https://github.com/hreinecke/sg3_utils/pull/22).

**USE AT YOUR OWN RISK.**

## Intended audience

Everyone who is willing to give this a try, in particular owners of less
common hardware (__SCSI tapes and medium changers__ most wanted).

## Running the test

Clone the [github repository](https://github.com/mwilck/test-udev-rules) 
or [download the tarball](https://codeload.github.com/mwilck/test-udev-rules/tar.gz/master).

Simply run the **test-scsi-rules** script on the system to test, and collect
the output file (the program prints the file name).

## Background information

The program installs the modified set of rules temporarily under
`/etc/udev/rules.d`, where it will __take precedence over any system-installed
rules__. If any existing files there would be overwritten, it aborts.
The program tries hard to clean up after itself.
You may want to double-check the contents of `/etc/udev/rules.d` after the
program has finished.

The test procedure runs `udevadm test` for all SCSI devices it detects in the
system, and stores the output. This procedure is repeated 3 times, for
different combinations of settings for the installed rules, and in the
unmodified system *before* installing the modified rules, and *after* removing
them again. Before each run of the test procedure, `systemd-udevd` is restarted.

This *should be safe* unless real udev events occur while the test is
running. But there are no guarantees. 

**USE AT YOUR OWN RISK.**

At the end, you'll get a `tar.gz` file with the program output. Please email
it to me for examination. The goal is to get identical results in
the test procedure in all runs, in particular for `ID_SERIAL` and other
`ID_xxx` variables, and for the symlinks that control how the system sees the
devices. Some minor deviations are normal. For ATA devices, results will
appear different in the cases with and without `sg3_utils`, because in the former
case they are treated as SCSI and in the latter as (S)ATA. The important
`ID_xxx` variables should be equal, though.

## I want to understand what this will do to my system!

That's very understandable. Please read the script. It just about 100 lines of
shell code.
