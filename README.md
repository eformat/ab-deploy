# Running A/B Deployments

###**How to do simple A/B deployments using manual scaling on OpenShift**

[See OpenShift Commons Briefing for original version](https://blog.openshift.com/openshift-3-demo-part-11-ab-deployments/)

Login to OpenShift WebConsole

Create a new project named **ab**

Create a new application named **app-a** using php builder image and this git repo:

    https://github.com/eformat/ab-deploy.git        master

Make sure to **not** deploy a route, and to add a **label**

    abgroupmember: true

Login to OpenShift using CLI and edit the deployment configuration to add this label to the **selector**

    oc edit dc/app-a

Expose the service using the selector label:

    oc expose dc/app-a --name=ab-service --selector=abgroupmember=true --generator=service/v1

Expose a route:

    oc expose service ab-service --name=ab-route --hostname=abdeploy.apps.foo.com

Scale it up:

    oc scale dc/app-a --replicas=4

Test it:

    for i in {1..10}; do curl http://abdeploy.apps.foo.com/; echo " "; done

Now deploy **app-b** from the branch, **label** it as well:

    https://github.com/eformat/ab-deploy.git        version2
    abgroupmember: true

Test it again, should see version 2 deployed now:

    for i in {1..10}; do curl http://abdeploy.apps.foo.com/; echo " "; done

Happy with new version, so scale it up:

    oc scale dc/app-b --replicas=2 && oc scale dc/app-a --replicas=2

Now remove app-a version 1 altogether:

    oc scale dc/app-b --replicas=4 && oc scale dc/app-a --replicas=0

Oh My God moment, revert back:

    oc scale dc/app-b --replicas=0 && oc scale dc/app-a --replicas=4

###**Success**

![Successful Deploy](http://eformat.co.nz/ose-training/images/ab-deploy.png)
