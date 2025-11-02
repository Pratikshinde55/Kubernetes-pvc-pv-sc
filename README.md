## Why need of PV in K8s?
  - As we know, we can easily launch pod and also we can easily do set-up, but most important is **Data**.
  - k8s provide by default **Empheral storage(temporary storage) volume**, it's means life of volume is equal to life of pod, if pod goes deleted then Data also deleted, So we use external volume by using PV.
  - So, make our data permenant or persistent we use **persistent volume[PV]** in k8s.

In Kubernetes we can achieve Persistent storage [PV] by two way that is Manual & Dynamic.
