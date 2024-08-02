# IGX Pull & Build Manual Test ![](https://img.shields.io/badge/draft-darkgoldenrod)
## Overview ![](https://img.shields.io/badge/draft-darkgoldenrod)
1. [Find an example to pull (From Ezra)](#step-1-find-an-example-to-pull-from-ezra)
2. [Create a docker image for running building Process](#step-2-create-a-docker-image-for-running-building-process)
3. [Create a container from the chosen image](#step-3-create-a-container-from-the-chosen-image)
3. Build the example
4. Run the example

## Step 1. Find an example to pull (From Ezra) ![](https://img.shields.io/badge/draft-darkgoldenrod)
I just have a call with Ezra and he said he has a repo on github, but I forgot which repo. So I am gonna go to [SST's GitHub](https://github.com/orgs/smartsurgerytek/repositories) to see if I can recall it.

```
Seeking...
```
I couldn't find it so I asked Ezra. He replied.
### Result
https://github.com/smartsurgerytek/holoscan_dentistry.git

## Step 2. Create a docker image for running building Process
I wanna know what kind of image I can use. So I first inspect the images I already have on IGX, via
```bash
docker images
```
And I found one
```
REPOSITORY  TAG             IMAGE ID        CREATED     SIZE
holohub     ngc-v2.0.0-igpu 22f1d9299a61    07/19/2024  9.58GB
```
I think this image is created by YUAN. It should be the best choice because it run all the build and launch job successfully before.

### Result
```
22f1d9299a61
```

## Step 3. Create a container from the chosen image
I need a container to run the building process. I have a [note](https://github.com/smartsurgerytek/Holohub-AWS-Setup#run-container) showing how to run an image.

The command is,
```bash
docker run -it --net host \
--gpus all \
-v $XAUTH:$XAUTH \
-v $XSOCK:$XSOCK \
-v $nvidia_icd_json:$nvidia_icd_json:ro \
-e XAUTHORITY=$XAUTH \
-e NVIDIA_DRIVER_CAPABILITIES=graphics,video,compute,utility,display \
-e DISPLAY \
--ipc=host \
--cap-add=CAP_SYS_PTRACE \
--runtime=nvidia \
--ulimit memlock=-1 \
-v /etc/group:/etc/group:ro \
-v /etc/passwd:/etc/passwd:ro \
-v $PWD:/workspace/holohub \
-w /workspace/holohub \
--group-add video \
$HOLOHUB_IMAGE
```
But this is for AWS, Ima change for a little bit, let's test some appoarches and see which one is runnable. I will list all the [appoarches to create a container](#appoarches-to-create-a-container) later, but we have to pass these commands to IGX. I'm currently use my PC and the best way to connect is [SSH into IGX](https://github.com/Smart-Surgery-Eason/Notes/blob/main/SSH%20Into%20IGX.md). I paste quick commands underneath, troubleshooting should contain in the document.
### Quick Commands for SSH Info IGX
On your PC, run:
```powershell
ssh smartsurgery@192.168.1.224
```
If not connected. Run this on IGX and reconnect again:
```bash
sudo systemctl restart ssh
```
### Appoarches to Create a Container
- [Appoarch 1: Change image ID](#appoarch-1-change-image-id)

### Appoarch 1: Change image ID
Literally change image Id from `$HOLOHUB_IMAGE` to `22f1d9299a61`,
```bash
docker run -it --net host \
--gpus all \
-v $XAUTH:$XAUTH \
-v $XSOCK:$XSOCK \
-v $nvidia_icd_json:$nvidia_icd_json:ro \
-e XAUTHORITY=$XAUTH \
-e NVIDIA_DRIVER_CAPABILITIES=graphics,video,compute,utility,display \
-e DISPLAY \
--ipc=host \
--cap-add=CAP_SYS_PTRACE \
--runtime=nvidia \
--ulimit memlock=-1 \
-v /etc/group:/etc/group:ro \
-v /etc/passwd:/etc/passwd:ro \
-v $PWD:/workspace/holohub \
-w /workspace/holohub \
--group-add video \
22f1d9299a61
```
**Result:**
```
docker run -it --net host --gpus all -v $XAUTH:$XAUTH -v $XSOCK:$XSOCK -v $nvidia_icd_json:$nvidia_icd_json:ro -e XAUTHORITY=$XAUTH -e NVIDIA_DRIVER_CAPABILITIES=graphics,video,compute,utility,display -e DISPLAY --ipc=host --cap-add=CAP_SYS_PTRACE --runtime=nvidia --ulimit memlock=-1 -v /etc/group:/etc/group:ro -v /etc/passwd:/etc/passwd:ro -v $PWD:/workspace/holohub -w /workspace/holohub --group-add video 22f1d9299a61
docker: invalid spec: ::ro: empty section between colons.
```
Then I ask ChatGPT, literally copy and paste the error, the conversation is [Here](ChatGPT/Docker%20Volume%20Error.md). It simply want me to run this on IGX:
```bash
echo $XAUTH
echo $XSOCK
echo $nvidia_icd_json
```
**Result:**
```

```
I feel I can simply remove `$XAUTH` and `$XSOCK` because I don't use X11-forward for this trial, but I have to investigate what `$nvidia_icd_json` is before I remove it. I also found that there are other `-v` flags which are
```bash
-v /etc/group:/etc/group:ro \
-v /etc/passwd:/etc/passwd:ro \
-v $PWD:/workspace/holohub \
```
A `-v` flag indicates `volume`, which means a share folder from our host machine to the container. Its syntax is
```bash
-v /absolute/path/on/your/host/machine:path/in/the/container:[ro]
```
The `ro` is optional, it means _readonly_.

With these knowledge, I have to check these paths too. Ima put the invesgation result in [anthor document](Investigation%20of%20nvidia_icd_json.md).

By the [result 1](Investigation%20of%20nvidia_icd_json.md#result-1) I try this first:

```bash
export nvidia_icd_json="/usr/lib/aarch64-linux-gnu/nvidia/nvidia_icd.json"
```

Another unsolve problem is `-v $PWD:/workspace/holohub`, it simply mount the `holohub` folder, which is the folder contains the **HoloHub Git repo**. I found that on my IGX machine, the Holohub Git repo is at `/home/smartsurgery/holoscan/holohub`, so Ima replace `$PWD` with it, as 
```
-v /home/smartsurgery/holoscan/holohub:/workspace/holohub
```
And then we can try the next appoarch.
### Appoarch 2: Solve `-v` errors

```bash
export nvidia_icd_json="/usr/lib/aarch64-linux-gnu/nvidia/nvidia_icd.json"
```
```
docker run -it --net host \
--gpus all \
-v $nvidia_icd_json:$nvidia_icd_json:ro \
-e XAUTHORITY=$XAUTH \
-e NVIDIA_DRIVER_CAPABILITIES=graphics,video,compute,utility,display \
-e DISPLAY \
--ipc=host \
--cap-add=CAP_SYS_PTRACE \
--runtime=nvidia \
--ulimit memlock=-1 \
-v /etc/group:/etc/group:ro \
-v /etc/passwd:/etc/passwd:ro \
-v /home/smartsurgery/holoscan/holohub:/workspace/holohub \
-w /workspace/holohub \
--group-add video \
22f1d9299a61
```
**Result**

The container was created, but with
```
Failed to detect NVIDIA driver version.
```
I tried `nvidia smi` in the container and it shows
```
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 540.3.1                Driver Version: N/A          CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  Orin (nvgpu)                  N/A  | N/A              N/A |                  N/A |
| N/A   N/A  N/A               N/A /  N/A | Not Supported        |     N/A          N/A |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```
It's identical to the host machine.


---
## References
- [SSH Into IGX Document](https://github.com/Smart-Surgery-Eason/Notes/blob/main/SSH%20Into%20IGX.md)
