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
- **RWO**: ReadWriteOnce (The volume can be mounted as **read-write by a Single node**)
- **ROX**: ReadOnlyMany (The volume can be mounted as **read only by many nodes**)
- **RWX**: ReadWriteMany (The volume can be mounted as **read-write by many nodes**)
- **RWOP**: ReadWriteOncePod (The volume can be mounted as **read-write by a Single** )

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

**Driver / Provisioner:**
 - If we want external persistent volume like AWS EFS S3 or Azure or other many; then we need to install driver of that external storage.

**Container storage interface[CSI]:**
 - CSI is just concept.
 - CSI helps to install driver/provisioner for SC. 
 
## In Kubernetes we can achieve Persistent storage [PV] by two way that is:
  1. Manual (By creating manually pvc pv)
  2. Dynamic (By using SC and pvc , pv dynamically created by SC)
  3. Dynamic (Create all pvc, pv and sc)


# 1 Dynamic PV :

i have used aws eks with single node cluster:

       eksctl create cluster  --name pscluster  --region ap-south-1  --version 1.30  --nodegroup-name psnodegp --instance-types t2.micro --nodes 3  --nodes-min 3  --nodes-max 6 --node-volume-size 8  --node-volume-type gp3  --ssh-access   --enable-ssm --instance-name psworkernode  --managed


**This image show the error that unable to launch pods in nodegroup:**
<img width="1691" height="921" alt="image" src="https://github.com/user-attachments/assets/4f91d01a-cd26-480b-91fc-2568db7c46a5" />


But, t2.micro only suuport 4 pods and system k8s pods are taken that space, so i need to scale-out nodegroup in my cluster:

      eksctl create nodegroup \
      --cluster=pscluster \
      --name=ps-demo \
      --node-type=t2.micro \
      --nodes=1 \
      --nodes-min=1 \
      --nodes-max=1 \
      --region=ap-south-1

    

before apply pvc there is no pv and pvc only sc :
<img width="1491" height="326" alt="image" src="https://github.com/user-attachments/assets/e8aafaca-b87e-4f4d-b55a-52becd1de7d7" />


After pvc claim no pv created because in SC "VOLUMEBINDINGMODE" is "WaitForFirstConsumer". So when any pod try use this pvc claim then only pv create and bound to the PV to pvc.
<img width="1491" height="785" alt="image" src="https://github.com/user-attachments/assets/db01ca5c-2781-4e12-bdf6-e1d8254669a2" />



