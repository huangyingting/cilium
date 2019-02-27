.. only:: not (epub or latex or html)

    WARNING: You are looking at unreleased Cilium documentation.
    Please use the official rendered version released here:
    http://docs.cilium.io

**************************
Installation on Google GKE
**************************

GKE Requirements
================

1. Install the Google Cloud SDK (``gcloud``)

::

   curl https://sdk.cloud.google.com | bash


For more information, see [Installing Google Cloud SDK](https://cloud.google.com/sdk/install)

2. Make sure you are authenticated to use the Google Cloud API:

::

   export ADMIN_USER=user@email.com
   gcloud auth login


The ``$ADMIN_USER`` will be used to create a cluster role binding

3. Create a project

::

   export GKE_PROJECT=gke-clusters
   gcloud projects create $GKE_PROJECT


4. Enable the GKE API for the project

::

   gcloud services enable --project $GKE_PROJECT container.googleapis.com

Create a GKE Cluster
====================

You can apply any method to create a GKE cluster. The example given here is
using the `Google Cloud SDK <https://cloud.google.com/sdk/>`_. This guide
will create a cluster on zone ``europe-west4-a``, feel free to change the zone
if you are in a different region of the globe.

.. code:: bash

    gcloud container --project $GKE_PROJECT clusters create cluster1 \
       --username "admin" --image-type COS --num-nodes 2 --zone europe-west4-a

When done, you should be able to access your cluster like this:

.. code:: bash

    kubectl get nodes
    NAME                                      STATUS   ROLES    AGE   VERSION
    gke-cluster1-default-pool-a63a765c-flr2   Ready    <none>   6m    v1.11.7-gke.4
    gke-cluster1-default-pool-a63a765c-z73c   Ready    <none>   6m    v1.11.7-gke.4

Create a cluster-admin-binding
==============================

.. code:: bash

    kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user $ADMIN_USER

Prepare the Cluster Nodes
=========================

By deploying the ``cilium-node-init`` DaemonSet, GKE worker nodes are
automatically prepared to run Cilium as they are added to the cluster. The
DaemonSet will:

* Mount the BPF filesystem
* Enable kubelet to operate in CNI mode
* Install the Cilium CNI configuration file

.. parsed-literal::

     kubectl create namespace cilium
     kubectl -n cilium apply -f \ |SCM_WEB|\/examples/kubernetes/node-init/node-init.yaml

Restart kube-dns
================

kube-dns is already running but is still managed by the original GKE network
plugin. Restart kube-dns to ensure it is managed by Cilium.

.. code:: bash

     kubectl -n kube-system delete pod -l k8s-app=kube-dns


Deploy Cilium + cilium-etcd-operator
====================================

The following all-in-one YAML will deploy all required components to bring up
Cilium including an etcd cluster managed by the cilium-etcd-operator.

.. tabs::
  .. group-tab:: K8s 1.13

    .. parsed-literal::

      kubectl apply -f \ |SCM_WEB|\/examples/kubernetes/1.13/cilium-with-node-init.yaml

  .. group-tab:: K8s 1.12

    .. parsed-literal::

      kubectl apply -f \ |SCM_WEB|\/examples/kubernetes/1.12/cilium-with-node-init.yaml

  .. group-tab:: K8s 1.11

    .. parsed-literal::

      kubectl apply -f \ |SCM_WEB|\/examples/kubernetes/1.11/cilium-with-node-init.yaml

  .. group-tab:: K8s 1.10

    .. parsed-literal::

      kubectl apply -f \ |SCM_WEB|\/examples/kubernetes/1.10/cilium-with-node-init.yaml

  .. group-tab:: K8s 1.9

    .. parsed-literal::

      kubectl apply -f \ |SCM_WEB|\/examples/kubernetes/1.9/cilium-with-node-init.yaml

  .. group-tab:: K8s 1.8

    .. parsed-literal::

      kubectl apply -f \ |SCM_WEB|\/examples/kubernetes/1.8/cilium-with-node-init.yaml


Restart remaining pods
======================

Once Cilium is up and running, restart all pods in ``kube-system`` so they can
be managed by Cilium, similar steps that we have previously performed for ``kube-dns``

::

    $ kubectl get pods --all-namespaces -o wide
    kube-system   event-exporter-v0.2.3-85644fcdf-9x77g                2/2     Running            0          30m   10.56.0.7     gke-cluster1-default-pool-a63a765c-flr2   <none>
    kube-system   fluentd-gcp-scaler-8b674f786-6vwfc                   1/1     Running            0          30m   10.56.0.2     gke-cluster1-default-pool-a63a765c-flr2   <none>
    kube-system   fluentd-gcp-v3.2.0-9ck4p                             2/2     Running            0          29m   10.56.1.6     gke-cluster1-default-pool-a63a765c-z73c   <none>
    kube-system   fluentd-gcp-v3.2.0-xhjwq                             2/2     Running            0          29m   10.56.0.10    gke-cluster1-default-pool-a63a765c-flr2   <none>
    kube-system   heapster-v1.6.0-beta.1-8f4db6558-vdqgg               2/3     CrashLoopBackOff   6          29m   10.56.1.4     gke-cluster1-default-pool-a63a765c-z73c   <none>
    kube-system   kube-dns-548976df6c-ckm2l                            4/4     Running            0          22m   10.56.1.245   gke-cluster1-default-pool-a63a765c-z73c   <none>
    kube-system   kube-dns-548976df6c-fz6gz                            4/4     Running            0          22m   10.56.0.151   gke-cluster1-default-pool-a63a765c-flr2   <none>
    kube-system   kube-dns-autoscaler-67c97c87fb-frvqj                 1/1     Running            0          30m   10.56.0.4     gke-cluster1-default-pool-a63a765c-flr2   <none>
    kube-system   kube-proxy-gke-cluster1-default-pool-a63a765c-flr2   1/1     Running            0          30m   10.164.0.2    gke-cluster1-default-pool-a63a765c-flr2   <none>
    kube-system   kube-proxy-gke-cluster1-default-pool-a63a765c-z73c   1/1     Running            0          30m   10.164.0.3    gke-cluster1-default-pool-a63a765c-z73c   <none>
    kube-system   l7-default-backend-7ff48cffd7-7qmv9                  1/1     Running            0          22s   10.56.0.69    gke-cluster1-default-pool-a63a765c-flr2   <none>
    kube-system   metrics-server-v0.2.1-fd596d746-x98pr                2/2     Running            0          29m   10.56.1.5     gke-cluster1-default-pool-a63a765c-z73c   <none>

You can choose to specify all pod names manually with ``kubectl delete pod -n kube-system <pod1> <pod2> ...``
or run ``kubectl -n kube-system delete pod --all``. This last option will also
restart ``kube-proxy`` pods which will not be managed by Cilium as those pods
are running in host network mode.

::

    $ kubectl -n kube-system delete pod --all
    pod "event-exporter-v0.2.3-85644fcdf-9x77g" deleted
    pod "fluentd-gcp-scaler-8b674f786-6vwfc" deleted
    pod "fluentd-gcp-v3.2.0-9ck4p" deleted
    pod "fluentd-gcp-v3.2.0-xhjwq" deleted
    pod "heapster-v1.6.0-beta.1-8f4db6558-vdqgg" deleted
    pod "kube-dns-548976df6c-ckm2l" deleted
    pod "kube-dns-548976df6c-fz6gz" deleted
    pod "kube-dns-autoscaler-67c97c87fb-frvqj" deleted
    pod "kube-proxy-gke-cluster1-default-pool-a63a765c-flr2" deleted
    pod "kube-proxy-gke-cluster1-default-pool-a63a765c-z73c" deleted
    pod "l7-default-backend-7ff48cffd7-7qmv9" deleted
    pod "metrics-server-v0.2.1-fd596d746-x98pr" deleted
