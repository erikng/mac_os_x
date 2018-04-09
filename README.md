Description
===========

# This cookbook is a fork of the original [mac_os_x cookbook](https://github.com/sous-chefs/mac_os_x) that has now been deprecated. This cookbook has been modernized so that your legacy cookbooks can continue to work while you migrate over to chef14's new macos_userdefaults built-in resource. Without this, you may not be able to deploy a single cookbook that can handle both resources while you migrate.

This cookbook has two resources for managing local user settings on OS
X:

* `mac_os_x_plist_file` - manages "`plist`" settings files for OS X applications.
* `mac_os_x_userdefaults` - manages settings in OS X's `defaults(1)` system.

This cookbook also includes a number of helper recipes.

Requirements
============

## Platform

Tested on macOS 10.13 and chef 13/14.

Resource/Provider
=================

## mac\_os\_x\_userdefaults

Manage the Mac OS X user defaults(1) system. The parameters to the
resource are passed to the defaults command and the parameters follow
convention of the OS X command. See the defaults(1) man page for
detail on how the tool works.

### Actions

- :write: write the setting to the specified domain. Default.

### Attribute Parameters

- domain: The domain the defaults belong to. Required. Name attribute.
- global: Whether the domain is global. Can be true or false. Default false.
- key: The preference key. Required.
- value: The value of the key. Required.
- type: Value type of the preference key.
- user: User for which to set the default.
- sudo: Set to true if the setting requires privileged access to modify. Default false.

`value` settings of `1`, `TRUE`, `true`, `YES` or `yes` are treated as
true by defaults(1), and are handled in the provider.

`value` settings of `0`, `FALSE`, `false`, `NO` or `no` are treated as
false by defaults (1) and are also handled by the provider.

### Limitations

The current version cannot handle plists or dictionaries.

### Examples

Simple example that uses the `com.apple.systempreferences` domain,
with a single key and value.

    mac_os_x_userdefaults "enable time machine on unsupported volumes" do
      domain "com.apple.systempreferences"
      key "TMShowUnsupportedNetworkVolumes"
      value "1"
    end

Specify a global domain. Note that the key is not required for global domains.

    mac_os_x_userdefaults "full keyboard access to all controls" do
      domain "AppleKeyboardUIMode"
      global true
      value "2"
    end

A boolean type that uses truthiness (TRUE).

    mac_os_x_userdefaults "finder expanded save dialogs" do
      domain "NSNavPanelExpandedStateForSaveMode"
      global true
      value "TRUE"
      type "bool"
    end

A setting that uses an int (integer) type.

    mac_os_x_userdefaults "enable OS X firewall" do
      domain "/Library/Preferences/com.apple.alf"
      key "globalstate"
      value "1"
      type "int"
    end

LWRP's can send notifications, so we can change the Dock, and then
refresh it to take effect.

    execute "killall Dock" do
      action :nothing
    end

    mac_os_x_userdefaults "set dock size" do
      domain "com.apple.dock"
      type "integer"
      key "tilesize"
      value "20"
      notifies :run, "execute[killall Dock]"
    end

This setting requires privileged access to modify, so tell it to use
sudo. Note that this will prompt for the user password if sudo hasn't
been modified for NOPASSWD.

    mac_os_x_userdefaults "disable time machine normal schedule" do
      domain "/System/Library/LaunchDaemons/com.apple.backupd-auto"
      key "Disabled"
      value "1"
      sudo true
    end

## mac\_os\_x\_plist\_file

Manages the property list (plist) preferences file with the
`cookbook_file` Chef resource. Files will be dropped in
`Library/Preferences` under the home directory of the user running
Chef.

### Actions

- :create: create the file. Default.

### Attribute Parameters

- source: file name to use in the files directory of the cookbook.
  Name attribute.
- cookbook: cookbook where the plist file is located.

### Examples

Write the iTerm 2 preferences to
`~/Library/Preferences/com.googlecode.iterm2.plist`.

    mac_os_x_plist_file "com.googlecode.iterm2.plist"

*Assumptions*

There are a couple glaring assumptions made by this recipe.

* If the domain starts with `/Library/Preferences`, then sudo is set
  to true, as that is not user writable.
* If the domain is `NSGlobalDomain`, then global is set to true.

License and Author
==================

* Author: Joshua Timberman (<cookbooks@housepub.org>)
* Author: Ben Bleything (<ben@bleything.net>)

* Copyright 2011-2013, Joshua Timberman

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
