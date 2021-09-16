---
title: Cluster Management
summary: Manage your cluster's schema, data, and more.
toc: true
---

<div class="filters clearfix">
    <a href="serverless-cluster-management.html"><button class="filter-button page-level current">{{ site.data.products.serverless }}</button></a>
    <a href="cluster-management.html"><button class="filter-button page-level">{{ site.data.products.dedicated }}</button></a>
</div>

This page describes the cluster management and cluster deletion workflows.

## Planning your cluster

{% include cockroachcloud/planning-your-serverless-cluster.md %}

## View Clusters page

On [logging in to the {{ site.data.products.db }} Console](https://cockroachlabs.cloud/), the **Clusters** page is displayed. The **Clusters** page provides a high-level view of your clusters.

For each cluster, the following details display:

- The cluster's **Name**
- The cluster's **Plan Type**, either Serverless or Dedicated
- The date and time the cluster was **Created**
- The cluster's current **State**
- The **Version** of CockroachDB the cluster is running
- The **Action** button, which is used to:
    - [**Delete cluster**](#delete-cluster)

To view and manage a specific cluster, click the name of the cluster. The [**Overview**](#view-cluster-overview) page will display.

## View cluster overview

The **Overview** page displays details about the selected {{ site.data.products.serverless }} cluster:

- The **Plan** that the cluster was created with
- The cluster's **Cloud** provider
- The cluster's **Region**
- The cluster's **Monthly Budget**
- The **Resource summary** of the cluster's budget allocation
- The cluster's **Storage usage**
- The cluster's **Request units used**
- The cluster's **SQL statements**

## Edit your spend limit  

You can edit your spend limit from the **Overview** page. The change in your spend limit will affect current and upcoming billing cycles. See [Planning your cluster](#planning-your-cluster) for more information on resource usage. 

1. Navigate to the **Overview** page for the cluster you want to edit.
1. Click the pencil icon in the top right corner of the **Monthly Budget** section.

    You will be taken to the **Edit cluster** page with a graph of your cluster's **Historical usage**.

1. In the **Spend limit** dropdown, enter your new budget.

    Your **Performance estimate** in the **Summary** sidebar will update accordingly.
    
1. Click **Update**.

## Restore data from a backup

Cockroach Labs runs full backups daily and incremental backups hourly for every {{ site.data.products.db }} cluster. The full backups are retained for 30 days and incremental backups for 7 days.

{{site.data.alerts.callout_info}}
All databases are not backed up at the same time. Each database is backed up every hour based on the time of creation. For larger databases, you might see an hourly CPU spike while the database is being backed up.
{{site.data.alerts.end}}

To restore your data, [contact us](https://support.cockroachlabs.com).

Additionally, you can [backup and restore](../{{site.versions["stable"]}}/take-full-and-incremental-backups.html) data on your own.

## Delete cluster

{{site.data.alerts.callout_danger}}
Deleting a cluster will delete all cluster data.
{{site.data.alerts.end}}

{{site.data.alerts.callout_info}}
Free {{ site.data.products.serverless-plan }} clusters are subject to deletion after 6 months of no activity.
{{site.data.alerts.end}}

Proceed with the following steps only if you are sure you want to delete a cluster:

1. Navigate to the **Overview** page for the cluster you want to delete.
1. Click the **Actions** button in the top right corner.
1. Select **Delete cluster**.
1. In the confirmation window, enter the name of the cluster.
1. Click **Delete**.
