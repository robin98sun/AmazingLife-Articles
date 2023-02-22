
# How does Kubernetes allocate CPU resource

Before reading this article, please prepare the knowledge of Kubernetes and `cgroup` on Linux.

Then, please refer to another article ["layer-by-layer Cgroup in Kubernetes"](https://medium.com/geekculture/layer-by-layer-cgroup-in-kubernetes-c4e26bda676c), which explains the title of this article very well.

When prepared, there is one thing left: how to map the Pods, or even the containers in a Pod, to the specific `cgroup` directory?

Let's get into the details which is quite simple.

When a Pod is deployed, no matter via CLI or via a programming library (e.g., the library for GoLang "k8s.io/client-go/kubernetes"), most likely we only have a name of the deployment as return. But, that name does not exist in the file system of `cgroup`.

[img:https://github.com/robin98sun/AmazingLife-Articles/raw/master/tutorials/resources/Screenshot%202023-02-22%20at%204.02.01%20PM.png]


