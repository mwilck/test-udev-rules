# test program for SCSI udev rules

This is a test program for the udev rules proposed for **systemd**
(https://github.com/systemd/systemd/pull/7594) and **sg3_utils**
(https://github.com/hreinecke/sg3_utils/pull/22).

**USE AT YOUR OWN RISK.**

Simply run the **test-scsi-rules** script on the system to test, and collect
the output file (see below).

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
running. But there are no guarantees, **USE AT YOUR OWN RISK.**

At the end, you'll get a `tar.gz` file with the program output. Please email
it to me for examination. The goal is to get identical results in
the test procedure in all runs, in particular for `ID_SERIAL` and other
`ID_xxx` variables, and for the symlinks that control how the system sees the
devices. Some minor deviations are normal. For ATA devices, results will
different in the cases with and without sg3_utils.
