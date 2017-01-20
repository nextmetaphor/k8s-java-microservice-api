# Java Microservice API on Kubernetes
Demonstrates how to quickly build and deploy a simple Java microservice API to a Kubernetes environment on a local macOS host. 

The `dockerized-java-microservice-api` is a git submodule referencing [https://github.com/nextmetaphor/dockerized-java-microservice-api](https://github.com/nextmetaphor/dockerized-java-microservice-api); this subsequently references [https://github.com/nextmetaphor/java-microservice-api](https://github.com/nextmetaphor/java-microservice-api) in the `java-microservice-api` directory; the base Docker image is a git module referencing [https://github.com/nextmetaphor/dockerized-alpine-java](https://github.com/nextmetaphor/dockerized-alpine-java) in the `base-docker-image` directory. Refer to these repositories for more information around how each is configured.

## Getting Started 
The instructions below detail the pre-requisite technologies; together with installation, deployment and validation steps for a simple Java-based microservice hosted on Kubernetes.

### Prerequisites
* [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* [Gradle](https://gradle.org/getting-started-gradle-java/#toggle-id-1)
* [Docker](https://www.docker.com/products/overview#/install_the_platform)

### Installing
#### Clone and Initialise the Repository
Clone this repository, then initialise the submodules; this needs to be done recursively.
```bash
$ git submodule update --init --recursive
```

#### Start the Kubernetes Environment
Follow [the instructions here](https://nextmetaphor.io/2017/01/19/local-kubernetes-on-macos/) to initialise a local macOS `minikube` environment.

#### Share the Docker Environment
The Kubernetes Docker daemon won't, by default, use the same daemon as used on the host macOS system. This is a pain when we build images on the host machine in that they won't be visible to Kubernetes. It's pretty straightforward to stand up a separate registry in the Kubernetes VM and manually push images; however, it's far easier to share the Kubernetes Docker daemon. See [the minikube documentation](https://github.com/kubernetes/minikube#reusing-the-docker-daemon) for more details.

To achieve this, it's as simple as running the command below:
 
 ```bash
 $ eval $(minikube docker-env)
 ```
 
 Examining the currently running Docker processes on the host macOS machine will reveal the Kubernetes components.
 ```bash
 $ docker ps
 
  CONTAINER ID        IMAGE                                                        COMMAND                  CREATED              STATUS              PORTS               NAMES
  41a9d6915fb6        gcr.io/google_containers/exechealthz-amd64:1.2               "/exechealthz '--c..."   About a minute ago   Up About a minute                       k8s_healthz.7bd33f3f_kube-dns-v20-m8zf8_kube-system_8dfb256b-de75-11e6-b7c8-62bffd1cd4e4_31548166
  410d0ddde58c        gcr.io/google_containers/kube-dnsmasq-amd64:1.4              "/usr/sbin/dnsmasq..."   About a minute ago   Up About a minute                       k8s_dnsmasq.a58c1183_kube-dns-v20-m8zf8_kube-system_8dfb256b-de75-11e6-b7c8-62bffd1cd4e4_d7accca6
  ca32a32406e7        gcr.io/google_containers/kubedns-amd64:1.9                   "/kube-dns --domai..."   About a minute ago   Up About a minute                       k8s_kubedns.4caf56e8_kube-dns-v20-m8zf8_kube-system_8dfb256b-de75-11e6-b7c8-62bffd1cd4e4_129d9bfc
  00538ba45c29        gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1   "/dashboard --port..."   About a minute ago   Up About a minute                       k8s_kubernetes-dashboard.d34bda63_kubernetes-dashboard-qhnvf_kube-system_8de5f5b1-de75-11e6-b7c8-62bffd1cd4e4_591df4a3
  242b7a08fdca        gcr.io/google-containers/kube-addon-manager:v6.1             "/opt/kube-addons.sh"    About a minute ago   Up About a minute                       k8s_kube-addon-manager.96c28b3c_kube-addon-manager-minikube_kube-system_014fb8f91f3d52450a942179a984bc15_69b6061b
  0b314a23f31f        gcr.io/google_containers/pause-amd64:3.0                     "/pause"                 About a minute ago   Up About a minute                       k8s_POD.d8dbe16c_kube-addon-manager-minikube_kube-system_014fb8f91f3d52450a942179a984bc15_774cc089
  e9c45912fee9        gcr.io/google_containers/pause-amd64:3.0                     "/pause"                 About a minute ago   Up About a minute                       k8s_POD.a6b39ba7_kube-dns-v20-m8zf8_kube-system_8dfb256b-de75-11e6-b7c8-62bffd1cd4e4_70bc51f9
  dd5058332b69        gcr.io/google_containers/pause-amd64:3.0                     "/pause"                 About a minute ago   Up About a minute                       k8s_POD.2225036b_kubernetes-dashboard-qhnvf_kube-system_8de5f5b1-de75-11e6-b7c8-62bffd1cd4e4_8fea42cd
 ```

#### Building the Java Microservice API Docker Image
From within the `dockerized-java-microservice-api` subdirectory of the repository, build the microservice using the provided script:

```bash
dockerized-java-microservice-api$ bash build.sh 

Sending build context to Docker daemon 17.92 kB
Step 1 : FROM alpine:latest
latest: Pulling from library/alpine
0a8490d0dfd3: Pull complete 
Digest: sha256:dfbd4a3a8ebca874ebd2474f044a0b33600d4523d03b0df76e5c5986cb02d7e8
Status: Downloaded newer image for alpine:latest
 ---> 88e169ea8f46
Step 2 : RUN apk fix
 ---> Running in 07c28757e3f1
WARNING: Ignoring APKINDEX.c51f8f92.tar.gz: No such file or directory
WARNING: Ignoring APKINDEX.d09172fd.tar.gz: No such file or directory
OK: 4 MiB in 11 packages
 ---> 88ae728c3e9e
Removing intermediate container 07c28757e3f1
Step 3 : RUN apk add openjdk8 --update-cache
 ---> Running in d95c2be3a67e
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/community/x86_64/APKINDEX.tar.gz
(1/36) Installing libffi (3.2.1-r2)
(2/36) Installing libtasn1 (4.9-r0)
(3/36) Installing p11-kit (0.23.2-r1)
(4/36) Installing p11-kit-trust (0.23.2-r1)
(5/36) Installing ca-certificates (20161130-r0)
(6/36) Installing java-cacerts (1.0-r0)
(7/36) Installing libxau (1.0.8-r1)
(8/36) Installing libxdmcp (1.1.2-r1)
(9/36) Installing libxcb (1.12-r0)
(10/36) Installing libx11 (1.6.4-r0)
(11/36) Installing libxcomposite (0.4.4-r0)
(12/36) Installing libxext (1.3.3-r1)
(13/36) Installing libxi (1.7.7-r0)
(14/36) Installing libxrender (0.9.10-r0)
(15/36) Installing libxtst (1.2.3-r0)
(16/36) Installing alsa-lib (1.1.3-r0)
(17/36) Installing libbz2 (1.0.6-r5)
(18/36) Installing libpng (1.6.25-r0)
(19/36) Installing freetype (2.7-r0)
(20/36) Installing libgcc (6.2.1-r1)
(21/36) Installing giflib (5.1.4-r1)
(22/36) Installing libjpeg-turbo (1.5.1-r0)
(23/36) Installing libstdc++ (6.2.1-r1)
(24/36) Installing openjdk8-jre-lib (8.111.14-r1)
(25/36) Installing java-common (0.1-r0)
(26/36) Installing krb5-conf (1.0-r1)
(27/36) Installing libcom_err (1.43.3-r0)
(28/36) Installing keyutils-libs (1.5.9-r1)
(29/36) Installing libverto (0.2.5-r0)
(30/36) Installing krb5-libs (1.14.3-r1)
(31/36) Installing lcms2 (2.8-r0)
(32/36) Installing pcsc-lite-libs (1.8.20-r0)
(33/36) Installing lksctp-tools (1.0.17-r0)
(34/36) Installing openjdk8-jre-base (8.111.14-r1)
(35/36) Installing openjdk8-jre (8.111.14-r1)
(36/36) Installing openjdk8 (8.111.14-r1)
Executing busybox-1.25.1-r0.trigger
Executing ca-certificates-20161130-r0.trigger
Executing java-common-0.1-r0.trigger
OK: 94 MiB in 47 packages
 ---> 3a41fe43c78c
Removing intermediate container d95c2be3a67e
Successfully built 3a41fe43c78c
Starting a Gradle Daemon (subsequent builds will be faster)
:clean
:compileJava
:processResources UP-TO-DATE
:classes
:findMainClass
:war
:bootRepackage
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 6.78 secs
Sending build context to Docker daemon 33.22 MB
Step 1 : FROM nextmetaphor/alpine-java:latest
 ---> 3a41fe43c78c
Step 2 : ADD java-microservice-api/build/libs/java-microservice-api.war /tmp/java-microservice-api.war
 ---> 8f49ae7eeaca
Removing intermediate container 55fce3078eeb
Step 3 : ENTRYPOINT java -jar /tmp/java-microservice-api.war
 ---> Running in 04d17e210f90
 ---> 9d40d7aaca37
Removing intermediate container 04d17e210f90
Successfully built 9d40d7aaca37
```

This will build a `nextmetaphor/alpine-java` Docker image, build the Java microservice code using gradle, then combine the two, building a `nextmetaphor/java-microservice-api` Docker image. 

## Deployment
From the root repository directory, create the replication controller, using the provided `yaml` file.
```bash
$ kubectl create -f k8s-definitions/java-microservice-api/java-microservice-api-1.0-rc.yaml
replicationcontroller "java-microservice-api-1.0-rc" created 
```
Similarly, create the service.
```bash
$ kubectl create -f k8s-definitions/java-microservice-api/java-microservice-api-svc.yml
service "java-microservice-api-svc" created
```
Examine these files for the specific configuration.

## Validation ##
Simply use `curl` to hit the exposed service on the specified `NodePort`.
```bash
$ curl -s http://$(minikube ip):30286/person/200 | json_pp
{
   "age" : 37,
   "homeAddressModel" : {
      "addressLine1" : "home-address-line-1",
      "town" : null,
      "postcode" : "postcode",
      "addressLine2" : "home-address-line-2",
      "houseNumber" : 1,
      "county" : "county",
      "addressLine3" : "home-address-line-3",
      "houseName" : "house-name",
      "country" : "country"
   },
   "id" : 200,
   "name" : "person-name",
   "workAddressModel" : null
}
```

## Licence ##
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
