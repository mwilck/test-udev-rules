# test program for SCSI udev rules

This is a test program for the udev rule changes I am proposing for 
[systemd/udev](https://github.com/systemd/systemd/pull/7594) and 
[sg3_utils](https://github.com/hreinecke/sg3_utils/pull/22). These two are
supposed to go together, but each can be used alone as well.

**USE AT YOUR OWN RISK.**

## Intended audience

Everyone who is willing to give this a try, in particular owners of less
common hardware (__SCSI tapes and medium changers__ most wanted).

## Running the test

**Dependencies:** standard system packages `systemd`,`udev`, `curl`, `tar`, and
`sg3_utils`. The latest versionis of `systemd` and `sg3_utils`, against which I made
the pull requests, are *not required*, as only the udev rules are tested. In
current Linux distributions, the system-provided versions of these packages
should be sufficient.

**Install:** Clone the [github repository](https://github.com/mwilck/test-udev-rules) 
or download and unpack the [tarball](https://codeload.github.com/mwilck/test-udev-rules/tar.gz/master).

**Run:** Simply run the **test-scsi-rules** script on the system to test, and collect
the output file (the program prints the file name).

### Test procedure

The program downloads the modified set of udev rules from github (internet
connection required), and installs them temporarily under
`/etc/udev/rules.d`, where they will __take precedence over any system-installed
rules__. If any existing rules files would be overwritten, the program refuses
to run.

The program tries hard to clean up after itself, even in case of errors.
You may want to double-check the contents of `/etc/udev/rules.d` after the
program has finished.

The test procedure runs `udevadm test` for all SCSI devices it detects in the
system, and stores the output. This procedure is repeated 4 times, for
different udev rule sets, and in the unmodified system *before* installing the
modified rules, and *after* removing them again.

### How dangerous is this?

This *should be safe*, as the test script itself runs `udevadm test` only, and
thus doesn't affect the running system.

The potential risk is __real udev CHANGE events__ occuring while the test is
running (unless you have 1000s of SCSI devices, the test should be finished in
less than a minute). If this happens, the system udev daemon uses the current
test rule set, and in very rare circumstances the results may differ in
significant ways from previous runs. In the worst case, the system may use the
device differently after the event (e.g. if `SYSTEMD_READY` changes). 
Thus it's recommended not to do e.g. SCSI device probing while this test is
running. **USE AT YOUR OWN RISK.** 

### I want to understand exactly what this will do to my system!

Please read the `test-scsi-rules` script. It just about 100 lines of shell code.

## Inspecting test results

At the end, you'll get a `tar.gz` file with the program output. Please email
it to me for examination. The goal is to get identical results in
the test procedure in all runs, in particular for `ID_SERIAL` and other
`ID_xxx` variables.

Use `check_results` script to get an overview over the test results. 
This script filters the output of the test program to provide
a quick summary of the differences between the test runs. It also prints
explanations for some expected errors.

### List of test scenarios

 - **00-before:** This is just the default system configuration.
 - **01-sysfs:** Uses the new rule set with 55-scsi-sg3_utils.rules,
   trying to read SCSI VPD pages from sysfs if possible.
 - **02-inquiry:** Similar to 01-sysfs, but without reading VPD pages
   from sysfs; sg_inq is called on the device nodes instead. The results
   should be almost identical to 01-sysfs.
 - **03-scsi_id:** Uses the new rule set with 55-scsi-sg3_utils.rules
   disabled, i.e. scsi_id based rules from 55-zz-scsi_id.rules
   are in effect.
 - **04-nosg3:** Uses the default system rule set with 55-scsi-sg3_utils.rules
   disabled, i.e. scsi_id based rules from 60-persistent-storage.rules
   are in effect. On systems that don't install 55-scsi-sg3_utils.rules
   by default (such as Fedora), this is identical to 00-before.
 - **05-after:** Exactly the same as 00-before. Used only to make sure no
   changes remain after the testing ends.
