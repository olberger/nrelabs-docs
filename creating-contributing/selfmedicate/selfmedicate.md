# Selfmedicate

In the last section, we started a Vagrant machine with all of the prerequisites installed for running the Antidote platform. In this section, we'll dive a little bit into the selfmedicate tool specifically, so you are able to use it to iterate on lesson content and quickly preview the changes you've made.

These instructions assume you've followed the previous section, and now have a running Vagrant machine, with an active SSH connection by running `vagrant ssh`. If not, please do that now before continuing.

Within the selfmedicate repo, the `selfmedicate.sh` script is our one-stop shop for managing the development environment. This script has several subcommands:

```text
./selfmedicate.sh -h
Usage: selfmedicate.sh <subcommand> [options]
Subcommands:
    start    Start local instance of Antidote
    reload   Reload Antidote components
    stop     Stop local instance of Antidote
    resume   Resume stopped Antidote instance

options:
-h, --help                show brief help
```

In **rare** circumstances, you might need to run the `start`, `stop`, or `resume` commands. However, these commands have been wired up to the Vagrant environment in such a way that this shouldn't be necessary.

The main command you'll probably want to run within the Vagrant environment is the `reload` subcommand. This allows Antidote to re-import curriculum content.

### Troubleshooting Self-Medicate

The vast majority of all setup activities are performed by the `selfmedicate` script. The idea is that this script shoulders the burden of downloading all the appropriate software and building is so that you can quickly get to focusing on lesson content. However, issues can still happen. This section is meant to direct you towards the right next steps should something go wrong and you need to intervene directly.

.. warning::

```text
The ``selfmedicate`` script is designed to make it easy to configure a local minikube environment
with everything related to Antidote installed on top. However, you'll always be well-served by
becoming familiar with ``minikube`` or even Kubernetes itself so that you are more able to troubleshoot
the environment when things go wrong.
```

Before asking for help, please first answer these questions:

* Have you read this page and the :ref:`Self-Medicate Vagrant <selfmedicate-vagrant>` doc in its entirety?
* Have you run a `git pull` on the latest `master` branch of the Self-Medicate and Curriculum repos?
* Have you looked at the :ref:`Common Self-Medicate Issues <common-selfmedicate-issues>` below?

If you answered "yes" to all of these questions, please `open an issue on the selfmedicate repository <https://github.com/nre-learning/antidote-selfmedicate/issues/new>`\_ and ensure you follow the instructions provided there.

The self-medicate tool comes with a `debug` subcommand which runs a series of commands for extracting information that will be needed for any troubleshooting activity. To run this subcommand and place all output in a file called `selfmedicatedebug.txt`, run the below:

.. CODE::

```text
./selfmedicate.sh debug > selfmedicatedebug.txt
```

When opening an issue on the Self-Medicate repository linked above, this information will be required to move forward with troubleshooting any issues, so please `create a new Github Gist <https://gist.github.com/>`\_ and paste the link into the relevant spot in your issue. **Please do not paste the debug contents into a Github Issue directly**.

.. \_common-selfmedicate-issues:

### Common Self-Medicate Issues

.. NOTE::

```text
All of the steps below assume a working :ref:`Vagrant <selfmedicate-vagrant>` environment, and a working
persistent connection to this virtual machine via ``vagrant ssh``.
```

Cannot Connect to the Web Front-End ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It's likely that the pods for running the Antidote platform aren't running yet. Try getting the current pods:

.. code::

```text
~$ kubectl get pods
NAME                                        READY   STATUS    RESTARTS   AGE
antidote-web-99c6b9d8d-pj55w                2/2     Running   0          12d
nginx-ingress-controller-694479667b-v64sm   1/1     Running   0          12d
syringe-fbc65bdf5-zf4l4                     1/1     Running   4          12d
```

You should see something similar to the above. The exact pod names will be different, but you should see the same numbers under the `READY` column, and all entries under the `STATUS` column should read `Running` as above.

In some cases the `STATUS` column may read `ContainerCreating`. In this case, it's likely that the images for each pod is still being downloaded to your machine. You can verify this by "describing" the pod that's not `Ready` yet:

.. code::

```text
kubectl describe pods -n=kube-system kube-multus-ds-amd64-ddxqc
Name:               kube-multus-ds-amd64-ddxqc
....truncated....
Events:
Type    Reason     Age   From               Message
----    ------     ----  ----               -------
Normal  Scheduled  19s   default-scheduler  Successfully assigned kube-system/kube-multus-ds-amd64-ddxqc to minikube
Normal  Pulling    17s   kubelet, minikube  pulling image "nfvpe/multus:latest"
```

In this example, we're still waiting for the image to download - the most recent event indicates that the image is being pulled. The `selfmedicate.sh` script has some built-in logic to wait for these downloads to finish before moving to the next step, but in case that doesn't work, this can help you understand what's going on behind the scenes.

If you're seeing something else, it's likely that something is truly broken, and you likely won't be able to get the environment working without some kind of intervention. Please `open an issue on the antidote-selfmedicate repository <https://github.com/nre-learning/antidote-selfmedicate/issues/new>`\_ with a full description of what you're seeing.

Lesson Times Out While Loading ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's say you've managed to get into the web front-end, and you're able to navigate to a lesson, but the lesson just hangs forever at the loading screen. Eventually you'll see some kind of error message that indicates the lesson timed out while trying to start.

This can have a number of causes, but one of the most common is that the images used in a lesson failed to download within the configured timeout window. This isn't totally uncommon, since the images tend to be fairly large, and on some internet connections, this can take some time.

There are a few things you can try. For instance, `kubectl describe pods <pod name>`, as used in the previous section, can tell you if a given pod is still downloading an image.

We can also use docker commands to check the list of docker images that have been successfully pulled:

.. code::

```text
docker image list
```
