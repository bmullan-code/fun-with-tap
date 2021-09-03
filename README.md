# fun-with-tap

### update/install carvel
```
 wget -O- https://carvel.dev/install.sh | bash
```

check versions
```
kapp version && ytt version && kbld version && imgpkg version
kapp version 0.39.0

Succeeded
ytt version 0.36.0
kbld version 0.30.0

Succeeded
imgpkg version 0.17.0
```

Deploy kapp controller
```
kubectl apply -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/latest/download/release.yml
```

Create the tap package
```
kubectl create ns tap-install

kubectl create secret docker-registry tap-registry \
-n tap-install \
--docker-server='registry.pivotal.io' \
--docker-username=TANZU-NET-USER \
--docker-password=TANZU-NET-PASSWORD


```


Create the package repository
```

apiVersion: packaging.carvel.dev/v1alpha1
kind: PackageRepository
metadata:
 name: tanzu-tap-repository
spec:
 fetch:
   imgpkgBundle:
     image: registry.pivotal.io/tanzu-application-platform/tap-packages:0.1.0 #image location
     secretRef:
       name: tap-registry
```

Deploy it

```
kapp deploy -a tap-package-repo -n tap-install -f ./tap-package-repo.yaml -y
```
       
Check it
```
tanzu package repository list -n tap-install

tanzu package repository list -n tap-install
/ Retrieving repositories...
  NAME                  REPOSITORY                                                         STATUS               DETAILS
  tanzu-tap-repository  registry.pivotal.io/tanzu-application-platform/tap-packages:0.1.0  Reconcile succeeded

```

list available packages
```
tanzu package available list -n tap-install
/ Retrieving available packages...
  NAME                               DISPLAY-NAME                              SHORT-DESCRIPTION
  accelerator.apps.tanzu.vmware.com  Application Accelerator for VMware Tanzu  Used to create new projects and configurations.
  appliveview.tanzu.vmware.com       Application Live View for VMware Tanzu    App for monitoring and troubleshooting running apps
  cnrs.tanzu.vmware.com              Cloud Native Runtimes                     Cloud Native Runtimes is a serverless runtime based on Knative
```

Check version
```

tanzu package available list cnrs.tanzu.vmware.com -n tap-install
/ Retrieving package versions for cnrs.tanzu.vmware.com...
  NAME                   VERSION  RELEASED-AT
  cnrs.tanzu.vmware.com  1.0.1    2021-07-30T15:18:46Z
  
```

Get the versions for each package
```
 tanzu package available list cnrs.tanzu.vmware.com -n tap-install
/ Retrieving package versions for cnrs.tanzu.vmware.com...
  NAME                   VERSION  RELEASED-AT
  cnrs.tanzu.vmware.com  1.0.1    2021-07-30T15:18:46Z

tanzu bmullan$ tanzu package available list appliveview.tanzu.vmware.com -n tap-install
/ Retrieving package versions for appliveview.tanzu.vmware.com...
  NAME                          VERSION  RELEASED-AT
  appliveview.tanzu.vmware.com  0.1.0    2021-09-01T00:00:00Z

tanzu bmullan$ tanzu package available list cnrs.tanzu.vmware.com -n tap-install
/ Retrieving package versions for cnrs.tanzu.vmware.com...
  NAME                   VERSION  RELEASED-AT
  cnrs.tanzu.vmware.com  1.0.1    2021-07-30T15:18:46Z

tanzu package available list accelerator.apps.tanzu.vmware.com -n tap-install
/ Retrieving package versions for accelerator.apps.tanzu.vmware.com...
  NAME                               VERSION  RELEASED-AT
  accelerator.apps.tanzu.vmware.com  0.2.0    2021-08-25T00:00:00Z
```

Install the packages
```
tanzu package available get PACKAGE-NAME/VERSION-NUMBER --values-schema -n tap-install

tanzu package available get cnrs.tanzu.vmware.com/1.0.1 --values-schema -n tap-install
tanzu package available get appliveview.tanzu.vmware.com/0.1.0 --values-schema -n tap-install
tanzu package available get accelerator.apps.tanzu.vmware.com/0.2.0 --values-schema -n tap-install
```

Install
```
# create the values files based on output from last 3 commands


tanzu package install cloud-native-runtimes -p cnrs.tanzu.vmware.com -v 1.0.1 -n tap-install -f cnr-values.yaml

tanzu package install app-accelerator -p accelerator.apps.tanzu.vmware.com -v 0.2.0 -n tap-install -f app-accelerator-values.yaml

tanzu package install app-live-view -p appliveview.tanzu.vmware.com -v 0.1.0 -n tap-install -f app-live-view-values.yaml


```

Verify packages are running
```
tanzu package installed list -n tap-install
| Retrieving installed packages...
  NAME                   PACKAGE-NAME                       PACKAGE-VERSION  STATUS
  app-accelerator        accelerator.apps.tanzu.vmware.com  0.2.0            Reconcile succeeded
  app-live-view          appliveview.tanzu.vmware.com       0.1.0            Reconcile succeeded
  cloud-native-runtimes  cnrs.tanzu.vmware.com              1.0.1            Reconcile succeeded
bmullan-a01:tanzu bmullan$
```
 
