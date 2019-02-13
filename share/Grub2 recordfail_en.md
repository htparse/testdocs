##Deactivate the GRUB2 recordfail Feature
GRUB2 comes with a feature that, after a failed boot attempt, during the next boot will automatically stop at the boot menu.

This may be undesirable in certain circumstances. Depending on the GRUB version, changes need to be made to either the file */etc/grub.d/00_header* or the variable GRUB_RECORDFAIL_TIMEOUT in the file */etc/default/grub*.

First, search for the following in the file */etc/grub.d/00_header*

```
if [ ${recordfail} = 1 ]; then
   set timeout=-1
else
  set timeout=${GRUB_TIMEOUT}
fi
```

If this is found (the exact wording), then simply add comment characters:

```
#if [ \${recordfail} = 1 ]; then
#    set timeout=-1
#else
    set timeout=${GRUB_TIMEOUT}
#fi
```

Alternatively, if you find the following:

```
if [ "\${recordfail}" = 1 ]; then
 set timeout=${GRUB_RECORDFAIL_TIMEOUT:--1}
else
 set timeout=${2}
fi
```

Then edit the file */etc/default/grub* and add the line

`GRUB_RECORDFAIL_TIMEOUT=5`

For a 5 second timeout.

In either case to update the configuration run

`update-grub`

Now GRUB2 will always use the set timeout.

###Manual reset of the recordfail value
The value for grubenv can be manually reset

`grub-editenv set recordfail=0`

or even removed

`grub-editenv unset recordfail`
