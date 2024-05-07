# How to move an app to another region - using flyctl

### This guide will provide the steps required to move an app from one region to another using the Fly.io CLI

## Step 1: check your existing configuration
Check the existing configuration of the app using the command below (from within the app root folder). The current region will be shown as **primary_region**.

`flyctl config show`

- Use `flyctl platform regions` to see the full list of region codes available for use in Fly apps.
- Use `flyctl regions list` to see the full list of regions being used across your app currently (it may not only be the configured primary region).

## Step 2: re-deploy app to the target region
Re-deploy the app specifying the new target region using the command below.

`flyctl deploy -r <region-code-string>`

This will re-build the app and its Machines using the new configuration for region. You can use the previous `config show` command to check that this has been updated.

- Tip: you might also choose to edit the **fly.toml** file directly to change the region. After this, execute `flyctl deploy` and skip Step 3.

## Step 3: update configuration settings
Replace the currently existing **fly.toml** configuration file (which is now out-of-date) with the new one by running the following.

`flyctl config save -a <existing-app-name>`

- Enter `y` when asked if you wish to overwrite the existing configuration file.

## Step 4: change region for existing Machines / Volumes
Although the previous steps change the region settings for the app, this does not affect any Machines (and their associated Volumes) that may have been already existing. The changes only take place for subsequently created Machines that are created using the app process group - see usage of [`flyctl scale count`](https://fly.io/docs/apps/scale-count/#scale-the-number-of-machines-in-a-single-region).

Read the steps below to see how to update the region settings for an existing Machine (and attached Volume).

`flyctl machines list`

`flyctl volumes list`

- These commands will show a list of the currently existing Machines and Volumes, and their associated regions. If there are no instances listed, or if all regions are as desired, you can stop here.
- Take note of the Machines list **ID** column and the Volumes list **Attached VM** column for use in the next commands. The attached VM of a Volume should correspond with the ID of a Machine.

`flyctl machine clone <machine-id> -r <region-code-string>`

- Clone the existing Machine using the above command, specifying the new target region. This will also clone any associated Volume.
- Use the previous `list` commands to check that the new Machine and Volume have been created with the correct region.

`flyctl machine stop <machine-id>`

`flyctl machine destroy <machine-id>`

`flyctl volume destroy <volume-id>`

- Use the above commands (the order is important) to stop and remove the old Machine and any attached Volume - as these have now been cloned to a different region.
- You can check the status of the app Machines using `flyctl status`.


_For more ways to use the Fly CLI, check out the [docs](https://fly.io/docs/flyctl/)._