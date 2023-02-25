
# How does Kubernetes allocate CPU resource

Before reading this article, please prepare the knowledge of Kubernetes and `cgroup` on Linux, for example ["How to manage cgroups with CPUShares"](https://www.redhat.com/sysadmin/cgroups-part-two#:~:text=The%20CPUShares%20value%20provides%20tasks,shares%20allocated%20to%20the%20cgroup).

Then, please refer to another article ["layer-by-layer Cgroup in Kubernetes"](https://medium.com/geekculture/layer-by-layer-cgroup-in-kubernetes-c4e26bda676c), which explains the title of this article very well.

When prepared, there is one thing left: how to map the Pods, or even the containers in a Pod, to the specific `cgroup` directories?

Let's get into the details which is quite simple.

When a Pod is deployed, no matter via CLI or via a programming library (e.g., the library for GoLang "k8s.io/client-go/kubernetes"), most likely we only have a name of the deployment as return. But, that name does not exist in the file system of `cgroup`, as the following two screenshots show:

![K3s deployments by name](https://github.com/robin98sun/AmazingLife-Articles/raw/master/tutorials/resources/Screenshot%202023-02-22%20at%204.02.01%20PM.png)

![cgroup CPU directory](https://github.com/robin98sun/AmazingLife-Articles/raw/master/tutorials/resources/Screenshot%202023-02-22%20at%204.02.17%20PM.png)

Notice: if the Pod was deployed with CPU resource requirements, e.g., limit and/or require, the root directory for the pod in `cgroup` is `/sys/fs/cgroup/cpu/kubepods`, otherwise, it is `/sys/fs/cgroup/cpu/kubepods/besteffort`.

Then, we notice there are sub-directories with the name `pod***` where `***` is like an UID. OK, let's see if there is the same thing in Kubernetes.

First, find the Pod of your interest. Then, describe it:
```
kubectl describe pods <pod name>
```
![](https://github.com/robin98sun/AmazingLife-Articles/raw/master/tutorials/resources/Screenshot%202023-02-22%20at%204.17.37%20PM.png)

There is a ContainerID for each container, use the part after `://` in the ContainerID to search in the `cgroup` directories, and that's where the container process located. Of cause, its parent directory is the place for the pod.


In your program, it is more easier to query the Pod UID together with the ContainerIDs of its containers using the Kubernetes library. The following is a reference code:
![](https://github.com/robin98sun/AmazingLife-Articles/raw/master/tutorials/resources/Screenshot%202023-02-22%20at%204.21.49%20PM.png)

There is one more thing you need to be careful: since the files in `cgroup` directories are mapped directly from the memory, their size can not be changed. But this does not mean they are immutable. They can be modified, but can only allocate one digital number for most of them, e.g., shares, quota, period, etc.. Here is an example for updating CPU quota in `cgroup`:
```python
def update_cpu_resources(resource_type, pod_path, value, period=100000):

    path = '/sys/fs/cgroup/cpu/' + pod_path + '/cpu.cfs_quota_us'

    with open(path, 'r+') as f:
        old_data = f.read()
        f.seek(0)
        f.write(str(value))
        f.truncate()
        # verify
        f.seek(0)
        new_data = f.read()
        print("old data: %d, new data: %d" % (old_data, new_data))
        f.close()
```

So far, you get all the information needed to bypass Kubernetes and operate the `cgroup` for Pods or containers on the fly. 

Fly safe and have fun!
