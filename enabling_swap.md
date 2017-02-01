# Enabling swap

If you have the cheapest DigitalOcean server with only 512mb of memory, you might run into some problems when running multiple programs or hosting many different web applications. You could upgrade to a server with more memory as DigitalOcean suggests, but you can also work around the problem by enabling swap. This guide is a shorter version of [How To Add Swap On Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04) guide found in DigitalOcean's community tutorials. You should read the original guide if you want to understand the process better and make some optimizations that are not included in this short version.

## Creating a swap file

Let's create a 1 gigabyte swap file located in `/swap`:

```
sudo fallocate -l 4G /swap
```

Check that it was created:
```
ls -lh /swap
```
Set permissions to 0600 so that it can only be accessed by root user:
```
sudo chmod 0600 /swap
```

## Enabling the swap file

We have now created the swap file and it's time to tell the system to set up the swap space:
```
sudo mkswap /swap
```

Enable the swap file:
```
sudo swapon /swap
```

Check that it is enabled:
```
sudo swapon -s

Filename                                Type            Size    Used    Priority
/swap                                   file            1048572 0       -1
```

Now we have enabled the swap file, but it will not be automatically enabled after a restart.

## Automatically enable swap on startup

Open up the file `/etc/fstab`:
```
sudoedit /etc/fstab
```

Add the following line to the bottom of the file:
```
/swap   none    swap    sw    0   0
```

Save the file and exit your editor. Swap should now be enabled automatically.
