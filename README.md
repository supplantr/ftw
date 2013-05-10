# for the watts

## dependencies

* bash
* udev
* hdparm
* iw

## configuration

The config file is located at __/etc/conf.d/ftw__.

### universal options

<dl>
  <dt>STATE_FILE</dt>
  <dd>File to which the state (adp or bat) will be written after the respective custom function (see below) is called.</dd>

  <dt>MODULES</dt>
  <dd>Array of modules to be removed when offline and inserted when online.</dd>

  <dt>PARTITIONS</dt>
  <dd>Device partitions affected by REMOUNT_OPTIONS.</dd>

  <dt>DEVICES</dt>
  <dd>Devices affected by BLOCKDEV_READAHEAD, HD_POWER_MANAGEMENT, and HD_SPINDOWN_TIMEOUT.</dd>
</dl>

### state-dependent options

<dl>
  <dt>CPUFREQ_GOVERNOR</dt>
  <dd>CPU power scheme governor.</dd>

  <dt>NMI_WATCHDOG</dt>
  <dd>Execute periodic NMI interrupts to monitor CPU lock ups?</dd>

  <dt>BUS_CONTROL</dt>
  <dd>Control everything in /sys/bus/*/devices/*. on means device should be resumed and autosuspend is not allowed. auto means device is allowed to autosuspend and autoresume</dd>

  <dt>USB_AUTOSUSPEND_TIMEOUT</dt>
  <dd>How long to wait before suspending USB devices.</dd>

  <dt>PCIE_ASPM_POLICY</dt>
  <dd>PCI Express Active State Power Management. Will probably require pcie_aspm=force kernel boot parameter.</dd>

  <dt>LAPTOP_MODE</dt>
  <dd>Attempt to maximize the amount of time disks spend in a low power state by submitting all future pending disk IO when performing an IO operation?</dd>

  <dt>DIRTY_RATIO</dt>
  <dd>Maximum percentage of memory dirty pages can occupy before processes are forced to write dirty buffers themselves.</dd>

  <dt>DIRTY_BACKGROUND_RATIO</dt>
  <dd>Maximum percentage of memory dirty pages can occupy before pdflush begins to write them.</dd>

  <dt>DIRTY_EXPIRE_CENTISECS</dt>
  <dd>How long data can be in the page cache before it expires, signifying it must be written at the next opportunity.</dd>

  <dt>DIRTY_WRITEBACK_CENTISECS</dt>
  <dd>How often pdflush wakes up to write data to disk.</dd>

  <dt>SCSI_HOST_POLICY</dt>
  <dd>Policy for SCSI host adapters.</dd>

  <dt>REMOUNT_OPTIONS</dt>
  <dd>Comma-separated list (option1,option2,etc) of mount options to be applied when remounting devices.</dd>

  <dt>BLOCKDEV_READAHEAD</dt>
  <dd>Block device readahead in kilobytes.</dd>

  <dt>HD_POWER_MANAGEMENT</dt>
  <dd>Hard drive Advanced Power Management. See hdparm(8) (-B flag section) for more information.</dd>

  <dt>HD_SPINDOWN_TIMEOUT</dt>
  <dd>How long to wait after disk activity stops before turning off hard drive's spindle motor. See hdparm(8) (-S flag section) for more information.</dd>

  <dt>SND_INTEL_POWER_SAVE</dt>
  <dd>Enable Intel HDA audio chipset power saving?</dd>

  <dt>SND_AC97_POWER_SAVE</dt>
  <dd>Enable AC97 audio chipset power saving?</dd>

  <dt>WIRELESS_POWER_SAVE</dt>
  <dd>Enable wireless adapter power saving?</dd>

  <dt>BACKLIGHT_BRIGHTNESS</dt>
  <dd>Dim backlight to save power? For max value (min is 0), check /sys/class/backlight/acpi_video*/max_brightness.</dd>
</dl>

### custom functions

The __custom-adp__ or __custom-bat__ function is called upon a change in state, before the state file is updated.

#### examples

Disable Wake-on-LAN on battery with [ethtool](https://www.kernel.org/pub/software/network/ethtool/):

    custom-bat() {
        ethtool -s eth0 wol d
    }

    custom-adp() {
        ethtool -s eth0 wol g
    }

Manage backlight brightness per state using [relight](http://xyne.archlinux.ca/projects/relight/):

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
