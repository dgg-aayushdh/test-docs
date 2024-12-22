# Debugging updates/build

## Debugging `gcp_boot_build`
1. During `gcp_boot_build` startup script is terminated without clear error message. At least once this occurred because `apt-get upgrade` installed a new version of `systemd`, which is then restarted. Since `systemd` runs the startup script it gets killed during the restart.

# Debugging a failed instance
## How to do it/where to look
Suppose you have an instance that never stopped running, or failed. The first place you should look is within that instance on google cloud.To do this: 
1. Navigate to the `VM instances` tab on google cloud and select the instance.
2. Click the `SSH` button and connect to the instance. A terminal-like window will pop up. 
3. The first thing to check is config.txt `cat /work/job/config.txt`. Verify that your parameters are what you expect them to be. 
4. Next take a look at the docker script log `cat /work/job/docker_script.log`. This can show you if/where your processes were interrupted. 
5. Next check out the gcp script log `cat /work/job/gcp_script.log`. This will show you the process your instance has progressed through.
6. Another possibility is that you have exceeded the instance's provided memory. To check this, run `htop`. THis will show you the memory and swap usage. You can increase the memory by changing the default machine used. The default is most likely `machine="n2-highcpu-32"`. Instead you can try `machine="n2-standard-8"` or if you're feeling spendy `machine="n2-highcpu-64"`
7. You can also check disk usage using `du -h`. If you are at/near 100% usage, you should consider changing the disk size. The default is most likely `work_disk="250G"` instead try `work_disk="500G"`


