# OCI Kubernetes and Logging Analytics

This project is an OCI Logging Analytics "Hello World" with Kubernetes.

This project is composed of:

- Hello World API app in Node.js (no db connectivity)
- Terraform scripts for Oracle Kubernetes Engine
- Deployment manifest for app with terraform helm_release.

## TODO

- Improves Fluentd config
- Query the logs in Logging Analytics
- Ingress controller with Load Balancer
- Destroy fails
- Add policies and Dynamic group
- Fix Ingress Controller
- Do we need parsers, entities, Sources specific for the hello-api app?

## Deploy from here

> You need to be administrator, for now. Working on enumerating policies required as an alternative.

[![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/vmleon/oci-hello-loganalytics/releases/download/v0.1.0/logan.zip)

## Enable access to Log Group with Instance Principal

> Requirement: Logging Analytics enabled on the OCI Region.

Create a Dynamic Group called `dynamic-group-oke-node-pool` that matches OKE node pool workers with matching rule:

```
All {instance.compartment.id = '<compartment_ocid>'
```

> You have to replace `<compartment_ocid>` for the compartment OCID where your Kubernetes Cluster is created.

Create a policy to allow access to Log Group with the following rule:

`Allow dynamic-group dynamic-group-oke-node-pool to {LOG_ANALYTICS_LOG_GROUP_UPLOAD_LOGS} in compartment <Logging Analytics LogGroup's compartment_name>`

## Logging Analytics Log Explorer

Go to **Menu** > **Observability & Management** > **Logging Analytics** > **Log Explorer**.

## Manual Test Application

List the helm release with `helm list`.

Follow the steps on the `helm get notes hello-api` to port-forwarding on localhost.

> WIP: Include ingress-nginx-controller

Test the application with `curl -s localhost:3000/hello`.

## Destroy

Before destroy you need to purge logs.

> FIXME: Can it be done with terraform? Not apparently

```
oci log-analytics storage purge-storage-data \
    -c $COMPARTMENT \
    --namespace-name $(oci os ns get --query 'data' | tr -d '\"') \
    --time-data-ended $(date -u +%FT%TZ)
```

> With Web Console:
>
> Go to **Menu** > **Observability & Management** > **Logging Analytics** > **Administration**.
>
> Go on the side menu to **Storage**.
>
> Click the red button **Purge Logs**.
>
> Select your Log Group Compartment.
>
> Click **Purge**.

Go to **Menu** > **Developer Services** > **Resource Manager** > **Stacks**.

Click on your stack.

Click on `Destroy`.

## Build the app (optional)

> You can use the following image publicly available `fra.ocir.io/fruktknlrefu/hello-api:latest` or build your own image. Jump to next section if you are reusing the public image.

Build the image yourself:

Inside folder `api` run the `podman build -t hello-api .`

Tag the image `podman tag hello-api:latest fra.ocir.io/TENANCY_NAMESPACE/hello-api:latest`. Change the URL for the region you are working, `fra`, `lhr`, etc.

Login the image registry with `podman login fra.ocir.io`. User is `TENANCY_NAMESPACE/YOUR_EMAIL` and the password is the Auth Token you can create for your user.

Push the image `podman push fra.ocir.io/TENANCY_NAMESPACE/hello-api:latest`.
