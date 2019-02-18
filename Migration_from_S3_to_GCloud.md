# Justification
All the content previously storaged at Amazon Webservices (AWS) was transferred to the Google Cloud (GCloud) Platform. The actions executed for this migration were: 
* Files and buckets from S3 transferred to GCloud Storage (keeping their original names)
* Instances from EC2 shut down and their disk images transferred to GCloud Storage. These disk images can be attached to a GCloud instance and used to recover all structure from the EC2 instance system (more details at [Attaching a disk from EC2 instance to a GCloud Instance](https://github.com/biobureaubiotech/gcloudManagement/wiki/Migration-from-S3-to-GCloud#attaching-a-disk-from-ec2-instance-to-a-gcloud-instance)). 
---

# Attaching a disk from an EC2 instance to a GCloud Instance
For some instances at EC2 we were intereted in keeping their system structure to use it in future work at GCloud (saving us time to restablish the system). This was achieved taking copies of their hard disks and storing them at GCloud Storage ([click here](https://github.com/biobureaubiotech/gcloudManagement/wiki/Migration-from-S3-to-GCloud#how-to-export-an-ec2-instances-disk) if you want to know how this was done). Those disks were stored at the bucket gs://s3-snapshots-disk-images (project: Bio Bureau Storage). In order to use those disks in a GCloud Instance, one should proceed with the following steps (assuming you are working at the Console graphical interface):
1. Transform the disk in a image. Go to Compute Engine > Images. Then hit the Create Image button. Choose a name for the instance and, at Source field, choose Cloud Storage File. After finding the disk object at GCloud Storage, hit the Create button to create the image. 
2. Once the disk image is created you can create a disk out of this image. Go to Compute Engine > Disks. Then hit the Create Disk button. Choose a name for the disk and, at Source Image (make sure Source Type field is selecting the Image option), search for the image recently created for the disk. You should be ready to hit the Create button and create the disk. 
3. Now you already have a disk ready to be attached to a GCloud instance. To attach it to the GCloud Instance ([why not to use it to immediately create a new GCloud instance?](https://github.com/biobureaubiotech/gcloudManagement/wiki/Migration-from-S3-to-GCloud#why-not-to-use-the-disk-created-from-an-ec2-instance-to-directly-create-a-new-gcloud-instance)), go to Compute Engine > VM Instances. Find the desidered instance and click on it. Then at the new page hit the Edit button. At Additional Disks, click on Add Item and select the desired disk. At the end of the page, hit Save and wait for the changes to be implemented. 
4. After attaching the disk to the instance it is time to mount the disk to start accessing it. To do that you will need to open your instance. Once the terminal is open, run the command: 
    ```console 
    username@instance:~$ lsblk
    ``` 
    It will show you all disks available. You will see that the instance already has a disk mounted (the one whose MOUNTPOINT is equals to "/") and another disk umounted. The umounted one is the disk you just attached. You will see that the disk already has a partition (named as "diskname+number", e.g. sdb1).     

    <p align="center">
      <img src="https://github.com/biobureaubiotech/gcloudManagement/blob/master/images/partition_to_mount_example.png">
    </p>

    It is the partition that will be used to mount the disk. Before mounting it, create a directory where the new disk will be kept. Now run the command:       
    ```console 
    username@instance:~$ sudo mount /dev/partition_name /path/to/directory/where/will/be/mounted
    ```
    It's done! Now you can access all the disk content at the directory chosen to hold the disk. 
---
# Extra    
### How to export an EC2 instance's disk?
Since AWS doesn't provide a tool for exporting their EC2 instance snapshots, we had to attach each existing snapshot to a new EC2 instance, create a disk image file for it (called disk.raw), compress it and export it. This was achieved after the following steps:   
1. create a EC2 volume from the EC2 snapshot  
2. attach the volume to an existing EC2 instance  
3. login to the EC2 instance's terminal
4. identify the name of the recently attached volume (using lsblk command, find the disk that is not mounted yet)  
5. launch the command:    
    ```console 
    username@instance:~$ sudo dd if=/dev/disk_name of=disk.raw
    ```
6. compress the disk image:
    ```console 
    username@instance:~$ sudo tar -Sczf name_of_compressed_disk-image.tar.gz disk.raw
    ```  
7. transfer compressed disk image to GCloud: gsutil cp name_of_compressed_disk-image.tar.gz gs://s3-snapshots-disk-images  
### Mount can be reverted if instance is stopped
Beware the possibility that if the instance is stopped, the disk recently attached and mounted can be umounted. If this happens, only run the mount command again. 
### Why not to use the disk created from an EC2 instance to directly create a new GCloud Instance? 
One could want to use the disk recently created to create a new GCloud Instance, instead of having to attach it to an existing instance and having the extra work of mounting the disk. However, it seems that EC2 disks are not properly configured to boot a new instance. So, whenever you try to use a disk imported from a EC2 to create a new GCloud instance, the instance fails to initialize. 
