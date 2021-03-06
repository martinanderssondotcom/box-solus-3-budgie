# A Vagrant box with Solus 3 Budgie

The artifact of this project is a manually packaged `.box` file with [Solus 3
Budgie][1] installed<sup>1</sup>.

Actually, the box is already packaged for you and distributed on
[Vagrant's website][2].

_This GitHub project_ is used as an issue tracker as well as a notepad of how
exactly the box was prepared. Except for setting up stuff like the Vagrant user
account and Vagrant's SSH access, nothing else has been added and nothing has
been removed to/from the Solus 3 Budgie distribution.

Make sure you have [Vagrant][3] installed, [VirtualBox][4] installed together
with the Extension Pack, then, in theory, all you should have to do in order to
get a Virtual Machine running with Solus 3 is:

    vagrant init pristine/solus-3-budgie
    vagrant up

<sub><sup>1</sup> The semantical concept captured here is elsewhere described as
a "minimal" and/or "base" box. I refrain from using either term since 3 GB with
a full office suite installed et cetera is hardly "minimal" nor am I convinced
all use-cases of this barebones box is to derive yet another box for
distribution as implied by the word "base". We are building a box. Period.</sub>

## Steps to reproduce the distributed box

### Create Virtual Machine

Create a new VM instance. Name it `solus-3-budgie`. Select type `Linux`, version
`Linux 2.6 / 3.x / 4.x (64-bit)`. Set memory size to `2048 MB`.

Notes

- 2 GB of RAM is a "system requirement [...] for an optimal experience"
  <sup>[[source][5]]</sup>.

![Create VM][img-01]

Smack in a dynamically allocated disk with max size `40 GB`, type `VMDK`.

Notes:

- 40 GB as limit seems to be what most people online use.
- VMDK is the final format used inside the exported box. It is okay to create
  the VM using another format. If so, then the disk will be converted to VMDK
  during export (original disk file remains intact). At first, I actually
  started out using a VDI which VirtualBox also knows how to "compact". But
  despite compacting the disk prior to export, the disk space occupied by the
  final box file was slightly larger compared to starting out with a VMDK. I
  have not researched if starting out with one of the other formats could yield
  further disk savings.

![Create VMDK disk][img-02]

Open settings. Enable a bidirectional clipboard.

Note:

- This will not take effect until way later after the installation of Guest
  Additions.

![Bidirectional clipboard][img-03]

Floppy disk?? There's nothing to be discussed here. Get rid of that shit.

![Disable boot-from floppy][img-04]

Enable 3D acceleration. Click "OK" to save the new settings and close the
dialog.

Notes:

- As "strongly recommended", by Solus "for better performance"
  <sup>[[source][6]]</sup>. However, be wary that the usual fix for VirtualBox
  issues related to graphics is to disable 3D acceleration =)
- 2D video acceleration is for Windows guests only <sup>[[source][7]]</sup>.
- No need to fiddle with the video memory; we shall invoke a bit of command line
  voodoo in the next step to bump it all the way to 256 MB - the GUI window
  pictured below only allows 128 MB.

![Enable 3D acceleration][img-05]

Using the terminal on your host machine, bump the video memory to `256 MB`:

    VBoxManage modifyvm solus-3-budgie --vram 256

Notes:

- On my Windows host, VBoxManage is located in
  `C:\Program Files\Oracle\VirtualBox`.
- I have not been able to decipher what effect - if any - various different
  video memory sizes have. I certainly do not know why VirtualBox limit the GUI
  control to 128 MB.

### Install OS

Mount the OS installation's ISO file (grab it [here][5]). You do that by
clicking on the little CD icon to the right in the next picture. Then select
"Choose Virtual Optical Disk File...".

![Mount OS installation CD][img-06]

Start the VM and he should get right into a desktop that has an "Install OS"
icon. Use that to start the installation.

![Select OS language][img-07]

![Find location automagically][img-08]

![Choose a keyboard layout][img-09]

![Choose your timezone][img-10]

![Choose disk][img-11]

![Disk configuration][img-12]

![Configure the bootloader & hostname][img-13]

The password is `vagrant`.

![Create the Vagrant user][img-14]

![Make Vagrant owner][img-15]

![Review installation options][img-16]

![Confirm yet again][img-17]

![Drink coffee while OS is installing][img-18]

**This is important**, at least it was for me: Do not click the "Restart now"
button!

When I click this button the VM will hang/freeze. Use the taskbar's shortcut
control to shut down the machine.

Yes, _shut down_ the machine, because in the next step we..

![Installation complete][img-19]

Unmount the installation medium.

![Unmount the installation CD][img-20]

### Prepare the box

Boot and log in.

![Log in][img-21]

If you get notifications to install updates, then don't. We will run a few shell
scripts that take care of that.

Open a terminal and type in:

    wget https://raw.githubusercontent.com/martinanderssondotcom/box-solus-3-budgie/master/prepare_box_part1.sh
    sudo sh prepare_box_part1.sh

While the first part is running, increase the system's audio volume to max and
enable autologin. Autologin can be enabled through the Users app (you must first
click the unlock icon in the top right corner before changes can be made to the
user).

**After the first script completes, reboot.**

Start Firefox. Type "about:preferences#privacy" into the address bar. Uncheck
"Allow Firefox to install and run studies" (AMAZING!!!).

Run:

    wget https://raw.githubusercontent.com/martinanderssondotcom/box-solus-3-budgie/master/prepare_box_part2.sh
    sudo sh prepare_box_part2.sh
    rm prepare_box_part1.sh
    rm prepare_box_part2.sh
    history -c

Note how you were asked to enter Vagrant's user password (`vagrant`) when you
executed the first part, but not when executing the second part. This is because
one of the things the first part did was to setup "passwordless sudo".

Essentially, the scripts will prepare the VirtualBox box for export- and
packaging into a Vagrant box. The scripts will not just setup passwordless sudo,
but also setup stuff like an SSH server and VirtualBox's Guest Additions. Which
version of Guest Additions to install is changeable by editing a variable at the
top of [part two][8].

![Solus terminal][img-22]

### Package the box

Download [this Vagrantfile][9] and put it in your working directory. Then do:

    vagrant package --base solus-3-budgie --output solus-3-budgie.box --vagrantfile Vagrantfile

Notes:

- If the machine is running, then Vagrant will attempt to shut it down before
  packaging starts.
- Box description- and version is specified during the box-upload process on
  [Vagrant's website][10].

[1]: https://solus-project.com
[2]: https://app.vagrantup.com/pristine/boxes/solus-3-budgie
[3]: https://www.vagrantup.com/
[4]: https://www.virtualbox.org/wiki/Downloads
[5]: https://solus-project.com/download/
[6]: https://solus-project.com/articles/software/virtualbox/en/
[7]: https://www.virtualbox.org/manual/ch04.html#guestadd-2d
[8]: https://github.com/martinanderssondotcom/box-solus-3-budgie/blob/master/prepare_box_part2.sh
[9]: https://github.com/martinanderssondotcom/box-solus-3-budgie/blob/master/Vagrantfile
[10]: https://app.vagrantup.com/boxes/new

[img-01]: screenshots/01-vb-create-vm.png
[img-02]: screenshots/02-vb-create-vmdk-disk.png
[img-03]: screenshots/03-vb-bidirectional-clipboard.png
[img-04]: screenshots/04-vb-disable-floppy-boot.png
[img-05]: screenshots/05-vb-enable-3d.png
[img-06]: screenshots/06-vb-mount-solus-iso.png

[img-07]: screenshots/07-os-language.png
[img-08]: screenshots/08-os-location.png
[img-09]: screenshots/09-os-keyboard.png
[img-10]: screenshots/10-os-timezone.png
[img-11]: screenshots/11-os-choose-disk.png
[img-12]: screenshots/12-os-disk-configuration.png
[img-13]: screenshots/13-os-bootloader-hostname.png
[img-14]: screenshots/14-os-user.png
[img-15]: screenshots/15-os-owner.png
[img-16]: screenshots/16-os-review.png
[img-17]: screenshots/17-os-confirm.png
[img-18]: screenshots/18-os-installing.png
[img-19]: screenshots/19-os-complete.png

[img-20]: screenshots/20-vb-unmount.png
[img-21]: screenshots/21-vm-login.png
[img-22]: screenshots/22-vm-terminal.png