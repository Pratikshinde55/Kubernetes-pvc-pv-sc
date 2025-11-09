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


# EKS cluster:

1. Create cluster without nodes first:

       eksctl create cluster \
       --name pscluster \
       --region ap-south-1 \
       --version 1.30 --without-nodegroup
   
2. Create DEFAULT Worker:s

        eksctl create nodegroup \
        --cluster pscluster \
        --region ap-south-1 \
        --name default-nodes \
        --node-type t3.small \
        --nodes 1 \
        --managed \
        --enable-ssm

 3. Create PS Dedicated Server Node (isolated server)

        eksctl create nodegroup \
        --cluster pscluster \
        --region ap-south-1 \
        --name ps-server-nodes \
        --node-type t3.small \
        --nodes 1 \  
        --managed \
        --node-labels node-type=ps \
        --enable-ssm

If labels to nodes not apply then run this command:

         kubectl label node ip-192-168-2-134.ap-south-1.compute.internal node-type=ps

 5. Taint PS Node (prevent other workloads):

        kubectl taint nodes -l node-type=ps dedicated=ps:NoSchedule

**What this does:**
- This taints the PS node so that NO other pods can run on it unless they explicitly tolerate that taint.
- -l node-type=ps => Select nodes with label node-type=ps
- taint nodes => Apply taint to those nodes
- dedicated=ps:NoSchedule => Block normal pods from scheduling there
- PS namespace pods can run there, Cluster system pods and other apps cannot run there.

 5. Create namespace and add to node label:

 <img width="1254" height="439" alt="image" src="https://github.com/user-attachments/assets/958d9cf4-41f2-42c9-86a5-5b54a2ab150a" />

 Annotations:  scheduler.alpha.kubernetes.io/node-selector: node-type=ps

 This means all pods in ps-namespace will automatically go to node-type=ps (my isolated server node). 
 This ensures any pod in ps-namespace will automatically schedule to PS node.

**Note:**
- Namespaces are NOT created on nodes
- Namespaces are Kubernetes logical spaces, not tied to a single node.
- we cannot create a namespace inside a node.

  
Note: don't use t2.micro otherwise this error show:
**This image show the error that unable to launch pods in nodegroup:**
<img width="1691" height="921" alt="image" src="https://github.com/user-attachments/assets/4f91d01a-cd26-480b-91fc-2568db7c46a5" />



# Install SC for gp3 EBS csi driver:


1. Check csi driver - ebs-csi-controller-  , ebs-csi-node- :

          kubectl get pods -n kube-system

2.  Downlaod ebs 

         curl -o ebs-csi-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-ebs-csi-driver/master/docs/example-iam-policy.json

3.  Create IAM policy for EBS:

        aws iam create-policy \
        --policy-name AmazonEKS_EBS_CSI_Driver_Policy \
        --policy-document file://ebs-csi-policy.json

<img width="1797" height="708" alt="image" src="https://github.com/user-attachments/assets/59bc438f-541a-4546-a064-aadaa172d7c9" />

4.   IAM OIDC provider enabled:

        eksctl utils associate-iam-oidc-provider \
        --region ap-south-1 \
        --cluster pscluster \
        --approve

<img width="1590" height="177" alt="image" src="https://github.com/user-attachments/assets/13779417-d31e-47ef-9bbd-6e9fc1685070" />
     
5.  Service account:

           eksctl create iamserviceaccount \
           --cluster pscluster \
           --namespace kube-system \
           --name ebs-csi-controller-sa \
          --attach-policy-arn arn:aws:iam::12345667890:policy/AmazonEKS_EBS_CSI_Driver_Policy \
          --approve \
          --override-existing-serviceaccounts \
          --role-name AmazonEKS_EBS_CSI_Driver_Role

 <img width="1873" height="510" alt="image" src="https://github.com/user-attachments/assets/30b17c8d-5f5a-4819-91b1-0569245ff315" />
         
6. verfiy version of addons:

         eksctl get addons --cluster pscluster

<img width="1891" height="340" alt="image" src="https://github.com/user-attachments/assets/92a75b05-9f4a-468e-8062-ea9a5b0734cd" />


7. create addon:

          eksctl create addon \
          --cluster pscluster \
          --name aws-ebs-csi-driver \
          --version v1.52.1-eksbuild.1 \
          --force \
          --service-account-role-arn arn:aws:iam::1234567890:role/AmazonEKS_EBS_CSI_Driver_Role
      
<img width="1872" height="317" alt="image" src="https://github.com/user-attachments/assets/34e0b13b-32d4-4825-9490-9f418ea35eb3" />

8 .add:

         aws iam attach-role-policy \
        --role-name eksctl-pscluster-nodegroup-default-NodeInstanceRole-gSsq8QpLqCEa \
        --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy

<img width="1288" height="485" alt="image" src="https://github.com/user-attachments/assets/61e4410d-d82f-4b9d-ac74-355ec2489055" />

9. Create SC:

    <img width="1752" height="768" alt="image" src="https://github.com/user-attachments/assets/a83cc6be-cc21-482d-94a7-362a63cd2508" />



# 1. Dynamic pv created by using PVC and SC: 
Here i use ebs driver for this demo, Please chcek how do set-up for ebs gp3 driver on AWS EKS.

1. Before creation set-up:
   <img width="1660" height="323" alt="image" src="https://github.com/user-attachments/assets/6f697946-134d-4bab-81f3-724a47a94e17" />

2. After apply pvc then pvc status become pending because the sc volumebinding is waitforfirstconsumer:

         kubectl apply -f my-app-pvc.yml

 <img width="1592" height="277" alt="image" src="https://github.com/user-attachments/assets/e961dc22-af6f-42a5-9d7b-5d77b4092a91" />

4. As soon as Pod use the pvc claim then pv created and bind to pvc:

        kubectl apply -f my-app.yml

   <img width="1890" height="460" alt="image" src="https://github.com/user-attachments/assets/812a27a0-be86-47eb-b0da-ae320f8fa6fb" />


5. pvc status show bound:

   <img width="1877" height="442" alt="image" src="https://github.com/user-attachments/assets/760fc7b5-1811-4af9-94dd-adf56f8ccd8a" />


6. Check how bound work:

   i am go inside that pods and go to mountPath and create index.html folder.
   <img width="1387" height="696" alt="image" src="https://github.com/user-attachments/assets/4ee8cade-4067-4380-bdab-2323e9ced008" />

7. Now check uid of pod:

          kubectl get pod my-app-67b8bbbfdd-pzkxq -n ps-namespace -o jsonpath='{.metadata.uid}'

   <img width="1421" height="60" alt="image" src="https://github.com/user-attachments/assets/a8bb9442-58de-4e26-9f1d-d78cf2c9c593" />

8. Now Go inside EC2 instance of that pods:

          kubectl get pods -n ps-namespace -o wide

   Then check at location: **/var/lib/kubelet/pods/7a99da45-5654-4d95-93d4-9d39b57e8cc9/volumes/kubernetes.io~csi/pvc-b6cd57b2-ea9a-41a4-8aa6-3f566fcedc30/mount**

   Here 7a99da45-5654-4d95-93d4-9d39b57e8cc9 this is pod uid and kubernetes.io~csi this is driver name and pvc-b6cd57b2-ea9a-41a4-8aa6-3f566fcedc30 this is pvc claim name.
   
9. check from Google:
    <img width="1043" height="208" alt="image" src="https://github.com/user-attachments/assets/8cda8920-af67-48d1-b59b-8c56b50d353b" />


# 2. PVC PV created manually:
In this type we don't use SC means directly use hostPath for my setup. We define pv and pvc manually and also created.


1. create PV and PVC manually:

           kubectl apply -f manual-pv-pvc.yml

   <img width="1905" height="577" alt="image" src="https://github.com/user-attachments/assets/46d21c33-22a3-401b-9349-a401f3acea5e" />

   In abhove image we can see that as sson we create pv and pv then they both bound at same time and not using any SC

2. Now Run my-app: [YAML File name: my-app-manual-pv-pvc.yml]

        kubectl apply -f my-app-manual-pv-pvc.yml

  <img width="1193" height="435" alt="image" src="https://github.com/user-attachments/assets/f0338e34-c20c-468a-94a7-1a1de1e95a45" />

3. Now test from EC2: On EC2 the hostPath store data at pv mentioned location that is **/data/myapp**

     <img width="1712" height="312" alt="image" src="https://github.com/user-attachments/assets/8066517d-78fb-412b-b4bd-5a8dd49e460a" />

