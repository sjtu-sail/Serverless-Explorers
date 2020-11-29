# Fission Research

### Architecture

````
Warning:

This architecture research is delicately detailed.
For presentation usage, please consider presents part of below:

1. All concepts in Basics
2. Controller, Executor, Build Manager and StorageSvc in Core Components
3. Logger and Message Queue Trigger in Optional Components
````

##### Basics

![image-20201128220556536](Fission%20Research/image-20201128220556536.png)

``Function`` is what to execute, written by user

``Environment`` is the function runtime, can be pre-compiled and specify what libraries to include

``Trigger`` is to call the specific function

``Archive`` is a zip file containing source code or compiled binaries.

``Deployment Archives`` are ``Archives`` with runnable functions in them

``Source Archives`` are those with source code in ``Deployment Archives`` 



##### Core Components

``Controller``

Deal with client requests

Accept REST API requests and create Fission resources

It contains CRUD APIs for functions, triggers, environments, Kubernetes event watches, etc

All fission resources are stored in Kubernetes ``CRDs``

![image-20201128221046628](Fission%20Research/image-20201128221046628.png)



``Executor``

Component to spin up function pods

Executor will check router's request

![image-20201128221258237](Fission%20Research/image-20201128221258237.png)

Executor have two type:

1. ``PoolManager``: initiate some ``generic pools for environments`` which are some pods have ``fetcher`` containers to dynamically load functions from ``CRDs``. This one used for short-living and short cold start time functions, not suitable for massive traffic. (cold start time usually less than 100ms)
2. ``NewDeployment``: creates a ``Kubernetes Deployment`` along with HPA for functions execution. Can specify resources requirement with minimum scale act like ``PoolManager``. Can handle massive traffic 

![Fig.2 PoolManager](Fission%20Research/poolmanager.png)

![image-20201129074150939](Fission%20Research/image-20201129074150939.png)

![Fig.3 NewDeploy](Fission%20Research/newdeploy.png)

![image-20201129074207794](Fission%20Research/image-20201129074207794.png)

![image-20201129074112308](Fission%20Research/image-20201129074112308.png)



``Router``:

``Router`` forwards HTTP requests to function pods. If there’s no running service for a function, it requests one from ``Executor``. It's can be scaled up for it's stateless.

![Fig.1 Router](Fission%20Research/router.png)

![image-20201129074425347](Fission%20Research/image-20201129074425347.png)



``Function Pod``:

``Function Pod`` is for serving HTTP requests from the clients. It contains two containers: ``Fetcher`` and ``Environment Container``

![Fig.1 Function Pod](Fission%20Research/function-pod.png)

![image-20201129074550106](Fission%20Research/image-20201129074550106.png)



``Build Manager``:

Watches the package & environments CRD changes and manages the builds of function source code. Once an ``environment`` that contains a builder image is created, ``Build Manager`` will then create ``Kubernetes Service`` and ``Deployment`` to build function source code into a usable resources. 

![Fig.1 Builder Manager](Fission%20Research/buildermanager.png)

![image-20201129075035991](Fission%20Research/image-20201129075035991.png)



``Builder Pod``

Is to build source archive into a deployment archive. Deployment archive is able to use in the ``function pod``. ``Builder Pod`` contains two containers: ``Fetcher`` and ``Builder Container``.

![Fig.1 Builder Pod](Fission%20Research/builder-pod.png)

![image-20201129075443348](Fission%20Research/image-20201129075443348.png)



``StorageSvc``

Home for all archives of packages with sizes larger than 256KB. The ``Builder`` pulls the source archive from the ``Storage Service`` and uploads deploy archive to it. The ``Fetcher`` inside the function pod also pulls the deploy archive for function specialization.

![Fig.1 StorageSvc](Fission%20Research/storagesvc.png)

![image-20201129075706502](Fission%20Research/image-20201129075706502.png)



##### Optional Components

Deployed as ``Kubernetes DaemonSet`` to help to forward function logs to a centralized database service (``influxDB``) for log persistence.

![image-20201129075816702](Fission%20Research/image-20201129075816702.png)



``KubeWatcher``:

Watches the ``Kubernetes API`` and invokes functions associated with watches, sending the watch event to the function (when a watch event occurs, it serializes the object and calls the function via the router). Doesn't have a reliable message bus.

![Fig.1 KubeWatcher Trigger](Fission%20Research/kubewatcher.png)

![image-20201129080014726](Fission%20Research/image-20201129080014726.png)



``Message Queue Trigger``: (Based on ``NATs`` and ``Kafka``)

Binds a ``message queue topic`` to a function:

Events from that ``topic`` cause the function to be invoked with the message as the body of the request. The trigger may also contain a response topic: if specified, the function’s output is sent to this response.

![Fig.1 Message Queue Trigger](Fission%20Research/mqtrigger.png)

![image-20201129081012968](Fission%20Research/image-20201129081012968.png)



``Timer``:

Works like ``Kubernetes CronJob`` but instead of creating a pod to do the task, it sends a request to ``router`` to invoke the function. It’s suitable for the background tasks that need to execute periodically.

![Fig.1 Timer Trigger](Fission%20Research/timer.png)

![image-20201129081349470](Fission%20Research/image-20201129081349470.png)

### Community

**Last Commit:  **

9 days ago  (up to 2020-11-28)



**Issue Status at last 30 days:**

6 issue closed, 5 still opened

3/5 issues are replied

4:5  reply ratio (replies num vers issue num)



**Fission Roadmap:**

It's empty, probably no feature to add at now



### Pro & Cons

**Pro:** 

It's Kube-native

Is Extensible: 

Python, NodeJS, Go, C#, PHP are officially supported, and support self-designed containers

It can be self designed, like keep-alive containers, HPA for routers, TLS.

It has high performance



**Cons:**

No draggable UI for multi serverless function call

It doesn't have a log monitor UI by default, you need to install Prometheus manually



**Not easy to use:**

You need to specify environment to include specific libraries manually

You need to specify minimum pods or function pool size manually