# ESXi WebUI

## Accessing the WebUI

  This one is too simple to warrant a step-by-step guide. Open the address (IP or DNS) of the ESXi server in a browser, log in (the root credentials set during install should work), and `<parodyHollywoodHackerVoice>you're in.</parodyHollywoodHackerVoice>`.

## Create a Network

1. In the sidebar, navigate to the **Networking** section.

2. Go to the **Virtual switches** tab, and click **Add standard virtual switch**

3. Name the switch appropriately, then, assuming you'll be using a dedicated VM to handle routing (running something like VyOS, PFsense, OpenWRT, et cetera), click the **x** button to delete **Uplink 1**.

4. Switch over to the **Port groups** tab, and click **Add port group**. Select the virtual switch created in the previous step.

## SSH access

1. In the sidebar, navigate to the **Host** section - this is the default section when loading the UI, so it moght already be connected.

2. In the **Actions** menu, hover over **Services** and click **Enable Secure Shell (SSH)**

## Add ISOs

1. SSH into the ESXi Host, and run the following to enable downloads:

```sh
esxcli network firewall ruleset set -r httpClient -e true
```

2. `cd` into a datastore directory (under `/vmfs/volumes`, and create a directory called `isos`, then `cd` into it.

3. Download the ISO files with `wget <URL>`. ESXi doesn't seem to trust any widely-used HTTP certificates out-of-the-box - attempting to run `wget https://www.google.com/robots.txt` results in the error "`wget: error getting response: Cannot assign requested address`". Passing the `--no-check-certificate` flag can bypass this, but obviously, extra caution is warranted when disabling certificate checking.

## Create a VM

1. In the sidebar, navigate to the **Virtual Machines** section. Click **Create / Register VM**. Fill out the presented fields.

2. When it comes time to configure hardware, make sure that the Network Adapter/s are set to the appropriate fields, and, unless you plan on network booting to the installer, also ensure that **CD/DVD drive 1** points to the installation ISO.
