# Setup instructions and User Access

This section provides an overview of where to find information and source code related to the deployment and administration of GFTS JupyterHub. It also explains how users can be added and by whom.

## GFTS Hub deployment

The source code and documentation to deploy the GFTS Hub on the DestinE Cloud Infrastructure (currently at OVH) are available in the `gfts-track-reconstruction` folder, located in the root directory.

A good starting point is the [README.md](https://github.com/destination-earth/DestinE_ESA_GFTS/blob/main/gfts-track-reconstruction/jupyterhub/README.md) file, located in the `gfts-track-reconstruction/jupyterhub` folder.

### GFTS Hub User Image

The user image is defined in the [gfts-track-reconstruction/jupyterhub/images/user](https://github.com/destination-earth/DestinE_ESA_GFTS/tree/main/gfts-track-reconstruction/jupyterhub/images/user) folder and consists of:

- [Dockerfile](https://raw.githubusercontent.com/destination-earth/DestinE_ESA_GFTS/main/gfts-track-reconstruction/jupyterhub/images/user/Dockerfile) where you can review the Pangeo base image we are using, and see how additional conda and pip packages are installed, among other details;
- [conda-requirements.txt](https://github.com/destination-earth/DestinE_ESA_GFTS/blob/main/gfts-track-reconstruction/jupyterhub/images/user/conda-requirements.txt) contains the list of packages to be installed with conda. To suggest the installation of a new conda package, you can edit this file and make a Pull Request. Be sure to follow the `requirements.txt` format when adding new packages.
- [requirements.txt](https://github.com/destination-earth/DestinE_ESA_GFTS/blob/main/gfts-track-reconstruction/jupyterhub/images/user/requirements.txt) contains the list of packages installed with pip. Similarly, you can suggest adding new pip packages by editing this file and submitting a Pull Request.

## S3 Buckets

Limited storage is available on the GFTS hub itself but we have setup different S3 buckets where we manage the data we need for the GFTS project. We currently have 3 different S3 buckets:

- "**gfts-reference-data**": This S3 bucket contains reference datasets (such as Copernicus Marine) that have been copied here to speed up access and data processing. Most GFTS users will only require read-only access to this bucket; a few GFTS users can write to it to copy new reference datasets. If you need additional reference datasets, please create an issue [here](https://github.com/destination-earth/DestinE_ESA_GFTS/issues/new).
- "**destine-gfts-data-lake**": This S3 bucket contains datasets generated by the GFTS, which are intended to be made public for all users with access to the GFTS hub;
- "**gfts-ifremer**": This S3 bucket contains datasets that are private to a specific group, in this case, private to IFREMER GFTS users. Users with access to this S3 bucket can read and write to it.

## Access to the GFTS Hub and S3 Buckets

### Getting access to the GFTS Hub and S3 Buckets

The first step is to create an [issue](https://github.com/destination-earth/DestinE_ESA_GFTS/issues/new) with the following information:

1. The GitHub username of the person you want to add to the GFTS Hub;
2. The list of S3 buckets this new person would need to access.
3. If a new group of users is required, please specify the name of the new S3 bucket to be created for this group and identify any existing users who need access to it. **A new group of users is only necessary if you have your own set of biologging data that must remain private and not be shared with everyone**.

:::{seealso}

The current list of authorized GFTS users can be found in [`gfts-track-reconstruction/jupyterhub/gfts-hub/values.yaml`](https://github.com/destination-earth/DestinE_ESA_GFTS/blob/12fa92d1a1e6f6f089a7bc8dbc26c8ed3f101b73/gfts-track-reconstruction/jupyterhub/gfts-hub/values.yaml#L150).

:::

### Giving access to the GFTS Hub and S3 Buckets (Admin only)

While everyone can initiate a Pull Request to add a new user, only a few administrators can grant access (especially write access) to S3 Buckets. Below are the steps to follow if you are an administrator:

1. Add the new user (github username) in **lowercase** in `gfts-track-reconstruction/jupyterhub/gfts-hub/values.yaml`;
2. Add the github username (lowercase) in `gfts-track-reconstruction/jupyterhub/tofu/main.tf`: adding the username to `s3_users` will grant readonly access to `gfts-reference-data` and read/write access to `destine-gfts-data-lake` S3 buckets. If the user needs write access to the IFREMER S3 bucket, add their username to `s3_ifremer_developers`. If the user only needs read access, add their username to `s3_ifremer_users` instead.
3. Run `tofu apply` to apply the S3 permissions. Ensure you are in the `gfts-track-reconstruction/jupyterhub/tofu` folder before executing the `tofu` command.
4. Update `gfts-track-reconstruction/jupyterhub/secrets/config.yaml` with the output of the command `tofu outpout -json s3_credentials_json`. This command needs to be executed in the `tofu` folder after applying the S3 permissions with `tofu apply`. If the file contains binary content, it means you do not have the rights to add new users to the GFTS S3 buckets and will need to ask a GFTS admin for assistance.
5. Don't forget to commit and push your changes!

Steps 3 and 4 are what actually grant the jupyterhub user s3 access.

:::{caution}

The following packages need to be installed on your system:

1. [ssh-vault](https://ssh-vault.com);
2. [git-crypt](https://github.com/AGWA/git-crypt/blob/master/INSTALL.md);
3. [opentofu](https://opentofu.org)

As an admin, you'll need to set up your environment. The GFTS maintainer will provide you with a key encrypted with your GitHub SSH key. Save the content sent by the GFTS maintainer into a file, and name it `ssh-vault.txt`. At the moment, the keys are known to [annefou](https://github.com/annefou) and [minrk](https://github.com/minrk).

```
cat ssh-vault.txt | ssh-vault view | base64 --decode > keyfile && git-crypt unlock keyfile && rm keyfile
```

Before executing the command above, ensure you have changed the directory to the root of the `DestinE_ESA_GFTS` git repository.

Thanks to the previous command, you should be able to `cat gfts-track-reconstruction/jupyterhub/tofu/secrets/ovh-creds.sh` and see a text file.

Finally to initialize your environment and execute `tofu` commands, you need to change the directory to the `gfts-track-reconstruction/jupyterhub/tofu` folder and source `secrets/ovh-creds.sh` e.g.:

```
source secrets/ovh-creds.sh
tofu init
tofu apply
```

:::

Then you are ready to go and can follow the steps explained above to grant access to S3 buckets to a new user.