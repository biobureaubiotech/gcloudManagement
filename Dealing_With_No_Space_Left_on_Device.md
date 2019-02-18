### The problem
Sometimes an instance disk gets full, and you cannot connect to it since the VM doesn't have space to create credential files.  

### The solution
Follow the steps to solve this issues:  
1. Dettach the disk from the instance:  
    ```console  
    username@machine:$ gcloud compute instances detach-disk name-of-instance --disk=name-of-disk  
    ```  
2. Create a temporary VM, that will be used to free up space on the dettached disk.  
3. Delete the instance whose disk was dettached at step 1  
4. Attach the disk dettached at step 1 to the temporary machine created at step 2. To do that, go to Compute Engine > VM Instances. Find the desidered instance and click on it. Then at the new page hit the Edit button. At Additional Disks, click on Add Item and select the desired disk. At the end of the page, hit Save and wait for the changes to be implemented.   
5. Connect to the temporary VM and mount the recently attached disk on it:  
    ```console 
    username@instance:~$ sudo mount /dev/partition_name /path/to/directory/where/will/be/mounted
    ```  
PS: [click here](https://github.com/biobureaubiotech/gcloudManagement/wiki/Dealing-with-No-Space-Left-on-a-Instance#the-partition_name) if you don't know what the partition_name is
6. Browse the folder used to mount the disk and delete/transfer files in order to free up some space.  
7. Umount the disk  
    ```console 
    username@instance:~$ sudo umount /dev/partition_name 
    ```  
7. Dettach the disk umounted at step 6 from the temporary VM:  
    ```console  
    username@machine:$ gcloud compute instances detach-disk name-of-instance --disk=name-of-disk  
    ```  
8. Create a new instance (it is possible to use the name of the original instance, since it has already been deleted) using the disk dettached at step 7 to boot.      


#### The partition_name  
At step 5, one may ask what exactly is the partition_name? To find it out, start by running the command: 
```console 
username@instance:~$ lsblk
``` 
It will show you all disks available. You will see that the instance already has a disk mounted (the one whose MOUNTPOINT is equals to "/") and another disk umounted. 
<p align="center">
   <img src="https://github.com/biobureaubiotech/gcloudManagement/blob/master/images/partition_to_mount_example.png">
</p>
The umounted one is the disk you just attached. And this disk has at least one partition (named as "diskname+number", e.g. sdb1). The main partition is the one whose size is equals to the size expected for the instance disk (usually dennoted by the number 1).      



 
