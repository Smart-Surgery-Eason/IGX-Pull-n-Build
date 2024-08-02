# Investigation of `nvidia_icd_json`
**Cause:** In [IGX Pull & Build Manual Test](README.md#appoarch-1-change-image-id), I try to create a container but failed. due to we lack of the `$nvidia_icd_json` variable. I feel this is out of the pulling and the building scope, so I create a seperate document to record everything about it.


Before I delve in, I want to have a bold guess, via checking my IGX mahcine files
```bash
sudo find / -name *nvidia*.json
```
**Result 1**
```
...
usr/lib/aarch64-linux-gnu/nvidia/nvidia_icd.json
...
/etc/vulkan/icd.d/nvidia_icd.json
...
```
There are many results so I trimed the iirelatives. Only `nvidia_icd.json` got my interest because the name matched with `nvidia_icd_json`.

Ima try to assign `usr/lib/aarch64-linux-gnu/nvidia/nvidia_icd.json` to `nvidia_icd_json` first and back to [IGX Pull & Build Manual Test](README.md#appoarch-1-change-image-id) due to the priority reason.
