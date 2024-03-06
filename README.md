# Hello world rust sample for prpl container

This sample use the prpl SDK to be compiled

## Community
prpl expertise center is available on mattermost https://mattermost.tech.orange/prpl

Find all communication on https://espace.agir.orange.com/display/ECPRPL/Resources+and+links

## Getting Started

To use this example with the prpl LCM SDK

```
devtool modify hello-world-rust
```
Then build the image
```
devtool build-image
```

### Prerequisites

This require the prpl SDK https://espace.agir.orange.com/display/ECPRPL/LCM+SDK+-+installation+and+usage


### Add code to SDK

Clone the git code to the SDK (this avoid proxy problem to access to internal gitlab)

Inside the SDK docker
```
cd workspace/source
git clone https://gitlab.tech.orange/prpl-ware/hello-world-rust.git
```

Here the `source` folder is used for the code source, by default yocto use the `sources` for external source, by difference this allow to separate local source from downloaded sources

### Create the recipe and activate the new process

Create a default recipe for the code
```
devtool add hello-world-rust /sdkworkdir/workspace/source/hello-world-rust
```

Edit this recipe in `workspace/recipes/` to specify the rust compiler. The `cargo.tml` of the project will be use
```
inherit cargo

SRC_URI = ""

SUMMARY = "Hello World in rust"

LICENSE = "CLOSED"
LIC_FILES_CHKSUM = ""

BBCLASSEXTEND = "native"
```
This recipe will compile with cargo, the cargo default recipe is used


A check can be done for the recipe in the image
```
devtool status
```

### Compilation

Run the compilation of the binarie
```
devtool build hello-world-rust
```

Check the build
```
ls -l tmp/pkgdata/container-cortexa53/ | grep hello-world-rust
```

Then build the entire image

```
devtool build-image
```
The binarie is in the image
```
ls -l /sdkworkdir/tmp/deploy/images/container-cortexa53/
```

## Deployement

The deployement on prpl use a docker registry then it is downloaded by prplOS

### Push to registry

```
skopeo copy oci:/sdkworkdir/tmp/deploy/images/container-cortexa53/image-lcm-container-minimal-container-cortexa53.rootfs-oci docker://prpl-registry.netguard.ovh/prpl/REGISTRYACCESS --dest-creds=<REGISTRYUSER>:<REGISTRYPASS>`
```

### Deploy on livebox

```
ubus-cli "SoftwareModules.InstallDU(URL ='docker://prpl-registry.netguard.ovh/prpl/REGISTRYACCESS', Username = <REGISTRYUSER>, Password = <REGISTRYPASS>, ExecutionEnvRef = 'generic', UUID = 'aade1eee-8ee1-5690-887f-b41aab7ca15e')"
```
**NB:** the UUID should respect the RFC-4122 v5 standard, online generator can be used.

Check the installation
```
lxc-ls -f
```

#### Retieve the container

Find the generated name (DUID) of the new container. Get the deployment unit number `SoftwareModules.DeploymentUnit.X`
```
ubus-cli "SoftwareModules.DeploymentUnit.?" | grep 'aade1eee-8ee1-5690-887f-b41aab7ca15e
> SoftwareModules.DeploymentUnit.5.UUID="aade1eee-8ee1-5690-887f-b41aab7ca15e"
```
Here it is **5**, check the DUID
```
ubus-cli "SoftwareModules.DeploymentUnit.5.?" | grep DUID
> SoftwareModules.DeploymentUnit.5.DUID="1c4f86b1-35ec-5a2c-997a-f1fa9271b8bf"
> SoftwareModules.DeploymentUnit.5.DUID="000000000000000000000000000000000000"
```
If the DUID is "000000000000000000000000000000000000", the container name still the given UUID, if not, the name is the DUID

This ID will be used for all commands

#### Start the process

Connect to the container
```
lxc-attach -n 1c4f86b1-35ec-5a2c-997a-f1fa9271b8bf
```

And run the new process
```
root@1c4f86b1-35ec-5a2c-997a-f1fa9271b8bf:/# hello-world-rust
Hello, Orange !
```

### Uninstall
To remove the container
```
ubus-cli 'SoftwareModules.DeploymentUnit.cpe-1c4f86b1-35ec-5a2c-997a-f1fa9271b8bf.Uninstall(RetainData = "No")'
```

## License

This project is licensed - see the [LICENSE](LICENSE.md) file for details.
