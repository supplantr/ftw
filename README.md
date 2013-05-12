# for the watts

## dependencies

* bash
* udev
* hdparm
* iw

## configuration

The config file is located at `/etc/conf.d/ftw`.

### universal options

**STATE_FILE**

 * File to which the state (`adp` or `bat`) will be written after the
   respective custom function (see below) is called.

**MODULES**

 * Array of modules to be removed when offline and inserted when online.

**PARTITIONS**

 * Device partitions affected by `REMOUNT_OPTIONS`.

**DEVICES**

 * Devices affected by `BLOCKDEV_READAHEAD`, `HD_POWER_MANAGEMENT`, and
   `HD_SPINDOWN_TIMEOUT`.

### state-dependent options

**CPUFREQ_GOVERNOR**

 * CPU power scheme governor.

**NMI_WATCHDOG**

 * Execute periodic NMI interrupts to monitor CPU lock ups?

**BUS_CONTROL**

 * Control everything in `/sys/bus/*/devices/*`.
 * `on` means device should be resumed and autosuspend is not allowed.
 * `auto` means device is allowed to autosuspend and autoresume.

**USB_AUTOSUSPEND_TIMEOUT**

 * How long to wait before suspending USB devices.

**PCIE_ASPM_POLICY**

 * PCI Express Active State Power Management.
 * Will probably require `pcie_aspm=force` kernel boot parameter.

**LAPTOP_MODE**

 * Attempt to maximize the amount of time disks spend in a low power state by
   submitting all future pending disk IO when performing an IO operation?

**DIRTY_RATIO**

 * Maximum percentage of memory dirty pages can occupy before processes are
   forced to write dirty buffers themselves.

**DIRTY_BACKGROUND_RATIO**

 * Maximum percentage of memory dirty pages can occupy before pdflush begins
   to write them.

**DIRTY_EXPIRE_CENTISECS**

 * How long data can be in the page cache before it expires, signifying it
   must be written at the next opportunity.

**DIRTY_WRITEBACK_CENTISECS**

 * How often pdflush wakes up to write data to disk.

**SCSI_HOST_POLICY**

 * Policy for SCSI host adapters.

**REMOUNT_OPTIONS**

 * Comma-separated list (`option1,option2,etc`) of mount options to be applied
   when remounting devices.

**BLOCKDEV_READAHEAD**

 * Block device readahead in kilobytes.

**HD_POWER_MANAGEMENT**

 * Hard drive Advanced Power Management.
 * See `hdparm(8)` (-B flag section) for more information.

**HD_SPINDOWN_TIMEOUT**

 * How long to wait after disk activity stops before turning off hard drive's
   spindle motor.
 * See `hdparm(8)` (-S flag section) for more information.

**SND_INTEL_POWER_SAVE**

 * Enable Intel HDA audio chipset power saving?

**SND_AC97_POWER_SAVE**

 * Enable AC97 audio chipset power saving?

**WIRELESS_POWER_SAVE**

 * Enable wireless adapter power saving?

**BACKLIGHT_BRIGHTNESS**

 * Dim backlight to save power?
 * For max value (min is `0`), check
   `/sys/class/backlight/acpi_video*/max_brightness`.

### custom functions

The `custom-adp` or `custom-bat` function is called upon a change in state,
before the state file is updated.

#### examples

Disable Wake-on-LAN on battery with
[ethtool](https://www.kernel.org/pub/software/network/ethtool/):

    custom-bat() {
        ethtool -s eth0 wol d
    }

    custom-adp() {
        ethtool -s eth0 wol g
    }

Manage backlight brightness per state using
[relight](http://xyne.archlinux.ca/projects/relight/):

    custom-bat() {
        if [[ -f $STATE_FILE && $(< $STATE_FILE) == 'adp' ]]; then
            relight save adp
            relight restore bat
        fi
    }

    custom-adp() {
        if [[ -f $STATE_FILE && $(< $STATE_FILE) == 'bat' ]]; then
            relight save bat
            relight restore adp
        fi
    }
