Nvidia official containers: <https://developer.nvidia.com/ai-hpc-containers>

Nvidia datacenter/cloud-native documentation: <https://docs.nvidia.com/datacenter/cloud-native/contents.html>

Architecture Overview: <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/arch-overview.html#arch-overview>

Potentially post on: <https://stackoverflow.com/questions/57066162/how-to-get-docker-to-recognize-nvidia-drivers>

## NVIDIA Container Toolkit
Installation guide for docker: <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker>

### Installation
Setup the package repository and the GPG key
```
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/experimental/$distribution/libnvidia-container.list | \
         sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
         sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

nvidia-container-toolkit package Installation
`$ sudo apt-get update`

`$ sudo apt-get install -y nvidia-container-toolkit`

BEFORE configuring the Docker daemon to recognize the NVIDIA Container Runtime, install the gpu driver-container 

## NVIDIA GPU driver-containers
Driver container catalog:
<https://catalog.ngc.nvidia.com/orgs/nvidia/containers/driver>

### Installation
You will need to update the NVIDIA Container Toolkit config file (`/etc/nvidia-container-runtime/config.toml`) so that the root directive points to the driver container:

`$ sudo sed -i 's/^#root/root/' /etc/nvidia-container-runtime/config.toml`

Verify the file's root directive
```
disable-require = false
#swarm-resource = "DOCKER_RESOURCE_GPU"
#accept-nvidia-visible-devices-envvar-when-unprivileged = true
#accept-nvidia-visible-devices-as-volume-mounts = false

[nvidia-container-cli]
root = "/run/nvidia/driver"
#path = "/usr/bin/nvidia-container-cli"
environment = []
#debug = "/var/log/nvidia-container-toolkit.log"
#ldcache = "/etc/ld.so.cache"
load-kmods = true
#no-cgroups = false
#user = "root:video"
ldconfig = "@/sbin/ldconfig.real"

[nvidia-container-runtime]
#debug = "/var/log/nvidia-container-runtime.log"
log-level = "info"

# Specify the runtimes to consider. This list is processed in order and the PATH
# searched for matching executables unless the entry is an absolute path.
runtimes = [
    "docker-runc",
    "runc",
]

mode = "auto"

    [nvidia-container-runtime.modes.csv]

    mount-spec-path = "/etc/nvidia-container-runtime/host-files-for-container.d"
```

Disable the Nouveau driver modules:

```
$ sudo tee /etc/modules-load.d/ipmi.conf <<< "ipmi_msghandler" \
  && sudo tee /etc/modprobe.d/blacklist-nouveau.conf <<< "blacklist nouveau" \
  && sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf <<< "options nouveau modeset=0"
```

Update the initramfs:

`$ sudo update-initramfs -u`

Configure the Docker daemon to recognize the NVIDIA Container Runtime:

`$ sudo nvidia-ctk runtime configure --runtime=docker`

In case you hit an error running the `nvidia-ctk` command and it looks something like this:
```
INFO[0000] Loading docker config from /etc/docker/daemon.json 
INFO[0000] Config file does not exist, creating new one 
INFO[0000] Flushing docker config to /etc/docker/daemon.json 
ERRO[0000] unable to flush config: unable to open /etc/docker/daemon.json for writing: open /etc/docker/daemon.json: no such file or directory
```

It means `docker` is installed as a snap package, and the daemon configuration is located at `/var/snap/docker/current/config/daemon.json`. Fix by creating a dummy file and copying the configuration back

```
$ sudo cp /var/snap/docker/current/config/daemon.json /etc/docker/daemon.json
$ sudo nvidia-ctk runtime configure --runtime=docker
$ sudo cp /etc/docker/daemon.json /var/snap/docker/current/config/daemon.json
```

Reboot your system (or VM) if required:

`sudo reboot`

### Running the driver container
```
$ sudo docker run --name nvidia-driver -d --privileged --pid=host -v /run/nvidia:/run/nvidia:shared -v /var/log:/var/log --restart=unless-stopped nvcr.io/nvidia/driver:525.85.12-5.15.0-69-generic-ubuntu22.04
```

Check the container's log, if the container is running properly it should look like this:
```
...
nvidia-drm.ko: OK
nvidia-modeset.ko: OK
nvidia-peermem.ko: OK
nvidia-uvm.ko: OK
nvidia.ko: OK
Setting up linux-modules-5.15.0-69-generic (5.15.0-69.76) ...
Processing triggers for linux-image-5.15.0-69-generic (5.15.0-69.76) ...
/etc/kernel/postinst.d/dkms:
 * dkms: running auto installation service for kernel 5.15.0-69-generic
```

You can try it out by running a cuda container with the `nvidia-smi` command to poll for the installed GPUs:

`$ sudo docker run --gpus all --rm nvcr.io/nvidia/cuda:11.8.0-base-ubuntu22.04 nvidia-smi`


### Create a container for later use using docker exec

```
$ sudo docker run --name cuda_11_8 -d -i -t --gpus all nvcr.io/nvidia/cuda:11.8.0-base-ubuntu22.04 /bin/sh
$ sudo docker exec -it cuda_11_8 sh
```
