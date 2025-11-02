## Why need of PV in K8s?
  - As we know, we can easily launch pod and also we can easily do set-up, but most important is **Data**.
  - k8s provide by default **Empheral storage(temporary storage) volume**, it's means life of volume is equal to life of pod, if pod goes deleted then Data also deleted, So we use external volume by using PV.
  - So, make our data permenant or persistent we use **persistent volume[PV]** in k8s.

## volumeMounts:
- If we want to make Pod data persistent then we mount volume.
- Mount means we add Pod local storage folder to external storage folder.
- Storage == Folder == file == Data
- This is concept is same like in Docker we mount volume while launch Docker container(-v /pratik:/var/www/html)

## Persistent Volume Claim[PVC]:
 If we want to use persistent volume to our pod, then we use pvc, that pvc we define only size of volume & access modes.

     kubectl get pvc

**Access Modes**:
- RWO: ReadWriteOnce (The volume can be mounted as read-write by a Single node)
- ROX: ReadOnlyMany (The volume can be mounted as read only by many nodes)
- RWX: ReadWriteMany (The volume can be mounted as read-write by many nodes)
- RWOP: ReadWriteOncePod (The volume can be mounted as read-write by a Single )

**GB & GI**
- GB == Gigabyte , 1GB = 10^9 bytes used by storage (for hard drive SSD)
- GI == Gibibyte 1Gi = 2^30 bytes used in computing for memory/storage size.
  
## Persistent Volume[PV]:
 PV means real volume, 1st we make PVC then we got PV.

     kubectl get pv

Note:
- **Without PVC(Persistent Volume Claim) we can't use PV.**
- PV needs PVC
- **one PVC only connect to only one PV at a time.**
- **one PV only connect to only one PVC at a time.**
- But, for a pod we can able to connect multiple different volumes means diff pv, so that need diff pvc.

## Storage Class[SC]:
SC is one who helps to make dynamic gain persistent storage. Storage Class need provisioner driver.

    kubectl get sc 

Driver / Provisioner:
 - If we want external persistent volume like AWS EFS S3 or Azure or other many; then we need to install driver of that external storage.

Container storage interface[CSI]:
 - CSI is just concept.
 - CSI helps to install driver/provisioner for SC. 
 
## In Kubernetes we can achieve Persistent storage [PV] by two way that is:
  1. Manual (By creating manually pvc pv)
  2. Dynamic (By using SC and pvc , pv dynamically created by SC)
  3. Dynamic (Create all pvc, pv and sc)



