# Live-0 Cluster Deprecation Notice

This guide refers to **live-0**, which is the name of our current Kubernetes cluster.

Live-0 is in the **eu-west-1** AWS availability zone, located in Dublin, Ireland.

Depending on the outcome of the Brexit process, it may become unacceptable to host some of our services outside of the UK. So, we are in the process of setting up a new cluster, **live-1** hosted in London (the **eu-west-2** AWS availability zone). We expect this cluster to be ready to host live services by **w/c 25/03/19**

This is also a good opportunity to rationalise certain aspects of our hosting platform, regardless of the Brexit outcome.

If you are using this guide to familiarise yourself with our hosting platform, and/or to set up non-production deployments of your service components for development or demonstration purposes, then please continue to use **live-0** as specified in this document.

However, please be aware that any production services should be deployed to **live-1**

Deploying to **live-1** in the first instance will avoid the need for another migration, after your service goes live.

If you have any questions, please ask in the **#ask-cloud-platform** slack channel.
