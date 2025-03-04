{{site.data.alerts.callout_danger}}
The CockroachDB Helm chart is compatible with Kubernetes versions 1.22 and earlier (the latest version as of this writing). However, no new feature development is currently planned. If you are experiencing issues with a Helm deployment on production, contact our [Support team](https://support.cockroachlabs.com/).

If you are already running a secure Helm deployment on Kubernetes 1.22 and later, you must migrate away from using the Kubernetes CA for cluster authentication. For details, see [Secure CockroachDB on Kubernetes](secure-cockroachdb-kubernetes.html?filters=helm#migration-to-self-signer).
{{site.data.alerts.end}}

{{site.data.alerts.callout_info}}
Secure CockroachDB deployments on Amazon EKS via Helm are [not yet supported](https://github.com/cockroachdb/cockroach/issues/38847).
{{site.data.alerts.end}}

1. [Install the Helm client](https://helm.sh/docs/intro/install) (version 3.0 or higher) and add the `cockroachdb` chart repository:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ helm repo add cockroachdb https://charts.cockroachdb.com/
    ~~~

    ~~~
    "cockroachdb" has been added to your repositories
    ~~~

1. Update your Helm chart repositories to ensure that you're using the [latest CockroachDB chart](https://github.com/cockroachdb/helm-charts/blob/master/cockroachdb/Chart.yaml):

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ helm repo update
    ~~~

1. The cluster configuration is set in the Helm chart's [values file](https://github.com/cockroachdb/helm-charts/blob/master/cockroachdb/values.yaml).

    {{site.data.alerts.callout_info}}
    By default, the Helm chart specifies CPU and memory resources that are appropriate for the virtual machines used in this deployment example. On a production cluster, you should substitute values that are appropriate for your machines and workload. For details on configuring your deployment, see [Configure the Cluster](configure-cockroachdb-kubernetes.html?filters=helm).
    {{site.data.alerts.end}}

    Before deploying, modify some parameters in our Helm chart's [values file](https://github.com/cockroachdb/helm-charts/blob/master/cockroachdb/values.yaml):

    1. Create a local YAML file (e.g., `my-values.yaml`) to specify your custom values. These will be used to override the defaults in `values.yaml`.

    1. To avoid running out of memory when CockroachDB is not the only pod on a Kubernetes node, you *must* set memory limits explicitly. This is because CockroachDB does not detect the amount of memory allocated to its pod when run in Kubernetes. We recommend setting `conf.cache` and `conf.max-sql-memory` each to 1/4 of the `memory` allocation specified in `statefulset.resources.requests` and `statefulset.resources.limits`.

        {{site.data.alerts.callout_success}}
        For example, if you are allocating 8Gi of `memory` to each CockroachDB node, allocate 2Gi to `cache` and 2Gi to `max-sql-memory`.
        {{site.data.alerts.end}}

        {% include_cached copy-clipboard.html %}
        ~~~ yaml
        conf:
          cache: "2Gi"
          max-sql-memory: "2Gi"
        ~~~

    1. For a secure deployment, set `tls.enabled` to `true`.

        {% include_cached copy-clipboard.html %}
        ~~~ yaml
        tls:
          enabled: true
        ~~~

        {{site.data.alerts.callout_info}}
        By default, the Helm chart will generate and sign 1 client and 1 node certificate to secure the cluster. To authenticate using your own CA, see [Secure the Cluster](secure-cockroachdb-kubernetes.html?filters=helm).
        {{site.data.alerts.end}}

1. Install the CockroachDB Helm chart, specifying your custom values file.

    Provide a "release" name to identify and track this particular deployment of the chart, and override the default values with those in `my-values.yaml`.

    {{site.data.alerts.callout_info}}
    This tutorial uses `my-release` as the release name. If you use a different value, be sure to adjust the release name in subsequent commands.
    {{site.data.alerts.end}}

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ helm install my-release --values {custom-values}.yaml cockroachdb/cockroachdb
    ~~~

    Behind the scenes, this command uses our `cockroachdb-statefulset.yaml` file to create the StatefulSet that automatically creates 3 pods, each with a CockroachDB node running inside it, where each pod has distinguishable network identity and always binds back to the same persistent storage on restart.

1. Confirm that CockroachDB cluster initialization has completed successfully, with the pods for CockroachDB showing `1/1` under `READY` and the pod for initialization showing `COMPLETED` under `STATUS`:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ kubectl get pods
    ~~~

    ~~~
    NAME                                READY     STATUS      RESTARTS   AGE
    my-release-cockroachdb-0            1/1       Running     0          8m
    my-release-cockroachdb-1            1/1       Running     0          8m
    my-release-cockroachdb-2            1/1       Running     0          8m
    my-release-cockroachdb-init-hxzsc   0/1       Completed   0          1h
    ~~~

1. Confirm that the persistent volumes and corresponding claims were created successfully for all three pods:

    {% include_cached copy-clipboard.html %}
    ~~~ shell
    $ kubectl get pv
    ~~~

    ~~~
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                                      STORAGECLASS   REASON    AGE
    pvc-71019b3a-fc67-11e8-a606-080027ba45e5   100Gi      RWO            Delete           Bound     default/datadir-my-release-cockroachdb-0   standard                 11m
    pvc-7108e172-fc67-11e8-a606-080027ba45e5   100Gi      RWO            Delete           Bound     default/datadir-my-release-cockroachdb-1   standard                 11m
    pvc-710dcb66-fc67-11e8-a606-080027ba45e5   100Gi      RWO            Delete           Bound     default/datadir-my-release-cockroachdb-2   standard                 11m    
    ~~~

{{site.data.alerts.callout_success}}
The StatefulSet configuration sets all CockroachDB nodes to log to `stderr`, so if you ever need access to a pod/node's logs to troubleshoot, use `kubectl logs <podname>` rather than checking the log on the persistent volume.
{{site.data.alerts.end}}
