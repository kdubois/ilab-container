# ilab-container
Run Instructlab from a container in Fedora 41 (with nvidia GPU support)

##Instructions:

Enable RPM Fusion Repos
`sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm`

## Install Nvidia Drivers

There may be extra steps for enabling secure boot.  View the following blog for further details: https://blog.monosoul.dev/2022/05/17/automatically-sign-nvidia-kernel-module-in-fedora-36/

`sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda`

Reboot to load new kernel drivers
`sudo reboot`

Check video driver
`lspci -n -n -k | grep -A 2 -e VGA -e 3D`

## Configure Podman with NVIDIA container runtime
To configure your machine running RHEL 9.4+, you can follow NVIDIA's documentation to install NVIDIA container toolkit and to configure Container Device Interface to expose the GPUs to Podman.

Here is a quick procedure if you haven't.

`curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
sudo dnf config-manager --enable nvidia-container-toolkit-experimental
sudo dnf install -y nvidia-container-toolkit`

Then, you can verify that NVIDIA container toolkit can see your GPUs.

`nvidia-ctk cdi list`

Finally, you can generate the Container Device Interface configuration for the NVIDIA devices:

`sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml`

## Run the GPU-accelerated ilab container
When running our model, we want to create some paths that will be mounted in the container to provide data persistence. As an unprivileged user, you will run a rootless container, so you need to let the internal user access the files in the host. For that, we use podman unshare chown.

`mkdir -p ${HOME}/.ilab`
`podman unshare chown 1001:1001 -R ${HOME}/.ilab`

Then, we can run the container, mounting the above folder and using NVIDIA GPUs.
`podman run --rm -it --user 1001 --device nvidia.com/gpu=all --ipc=host --network host --volume ${HOME}/.ilab:/opt/app-root/src:Z --entrypoint ilab quay.io/rh-aiservices-bu/instructlab-workbench-code-server-cuda:0.20.1 config init`

To make things easier, we can create an alias (and add it to our ~/.bashrc file)
`alias ilab="podman run --rm -it --user 1001 --device nvidia.com/gpu=all --ipc=host --network host --volume ${HOME}/.ilab:/opt/app-root/src:Z --entrypoint ilab quay.io/rh-aiservices-bu/instructlab-workbench-code-server-cuda:0.20.1"`

And now we can just use the 'ilab' command:
`ilab config init`





