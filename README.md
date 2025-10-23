# ATTENTION

This role only configures snapper.
Installation of snapper must be done manually:

```sh
sudo apt install snapper
```

The default way that snapper works is to automatically create a new subvolume _“.snapshots”_ under the path of the subvolume that we are creating a snapshot. Because we want to keep our snapshots separated from the backed up subvolume itself we must remove the snapper created _“.snapshot”_ subvolume and then re-mount using the one that we created before in a separate subvolume at _@snapshots_.

```sh
# move our subvolume & mountpoint out of the way
cd /
sudo umount .snapshots  
sudo rm -r .snapshots

# Now we can create a new configuration for Snapper with:
sudo snapper --config root create-config /
```

This new configuration should have created a new _“.snapshots”_ directory and as well a new BTRFS subvolume of the same name. We will remove this new subvolume and link our own created _@snapshots_ subvolume to this path, so our snapshots will be safely stored in a different location.

To remove the auto-created subvolume use:

```sh
# delete the subvolume created by Snapper:
sudo btrfs subvolume delete /.snapshots
# re-create the directory:
sudo mkdir /.snapshots
# re-mount our @snapshots to “/.snapshots” with:
sudo mount -av
```

Snapper is ready to be used, now we can start to set the configuration that we want for snapper. Snapper can automatically create new snapshots on a schedule, on every boot, before and after installing a new package.

# EXAMPLES

```yml
# fk
- ansible.builtin.import_role:
		name: snapper
	vars:
		snapper:
			configs:
				add:
				- fk/opt
				- fk/root
			systemd:
				cp:
				- snapper-root-writable.service
				- snapper-root-writable.timer
				enable:
				- snapper-root-writable.timer
			timeline:
				timer:
					enabled: false
			boot:
				timer:
					enabled: false
```