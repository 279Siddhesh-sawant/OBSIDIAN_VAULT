### Q. 1 Enumerate the NFS service and submit the contents of the flag.txt in the "nfs" share as the answer.
![](Images/nfs_1.png)
 Make a directory and mount shares to it.
 ```
 mkdir target-nfs
 ```
![](Images/nfs_2.png)

 ```
 sudo mount -t nfs 10.129.202.5:/ target-nfs -o nolock
 ```

![](Images/nfs_3.png)

### Q.2 Enumerate the NFS service and submit the contents of the flag.txt in the "nfsshare" share as the answer.

![](Images/nfs_4.png)
