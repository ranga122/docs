---
title: Cluster Virtualization Overview
summary: Learn more about cluster virtualization and how it facilities cluster-to-cluster replication.
toc: true
docs_area: deploy
---

This page gives an overview of _cluster virtualization_ in CockroachDB {{page.version.version}}. Cluster virtualization allows you to separate a cluster's _control plane_ from its _data plane_. The cluster's control plane manages cluster nodes and node-to-node traffic, while its data plane reads and writes data to the cluster's storage. Cluster virtualization must be enabled to configure cluster-to-cluster replication{% comment %}TODO: Link{% endcomment %}.

When cluster virtualization is disabled, the `cockroach` process on a node runs a single cluster that handles all system and user activity, and manages the cluster's control plane and data plane. When cluster virtualization is enabled, the `cockroach` process on a node runs both a _system interface_ and one or more _virtual clusters_.

- The system interface manages the cluster's control plane. Administrative access to the system interface can be restricted. Certain low-level cluster settings can be modified only on the system interface.
- One or more virtual clusters manage their own data plane. Administrative access on a virtual cluster does not grant any access to the system interface. The effect of some settings is scoped to the virtual cluster rather than the system interface.

## Differences when cluster virtualization is enabled

When cluster virtualization is enabled, CockroachDB's behavior changes in several key areas, as described in the following sections.

### Backup and restore

The scope of backup/restore is, by default, scoped to a virtual cluster level with a multi tenant architecture. This means that:
Backups taken from a virtual cluster will only backup data for that virtual cluster
In order to take a backup of the entire cluster, you must do so from within the system tenant and include the includes_secondary_tenant flag (this is the default in serverless but not self-hosted)
We expect backups taken by an application to only be scoped to their virtual cluster, additionally these backups can be restored as a new virtual cluster in any cluster


### Cluster logs

When cluster virtualization is enabled, cluster log messages for the system interface and each virtual cluster are labeled according to the cluster they are associated with.

### SQL API

When cluster virtualization is enabled, certain low-level SQL APIs (TODO: Which ones) are accessible only by a virtual cluster's cluster administrators, and not by other cluster users.

### Span Config Bounds

- Span config bounds can be set at the overall cluster level or in the system interface. Span config bounds set in the system interface override zone config settings that are set in a virtual cluster.
- Zone configs can be set only in a virtual cluster, but span config bounds set in the system interface override zone config settings that are set in a virtual cluster.

### DB Console

In the DB Console, users with access to both the system interface and virtual clusters can select which cluster to connect to.

Some pages and views are only viewable from the system interface, including those pertaining to overall cluster health.

When connected to a virtual cluster, most pages and views are scoped to the virtual cluster. by default the DB Console displays only metrics about that virtual cluster, and excludes metrics for other virtual clusters and the system interface. To allow the DB Console to display system-level metrics from within a virtual cluster, you can grant the virtual cluster the `can_view_node_info` permission.

## Cluster settings scoped to the virtual cluster

TODO: https://docs.google.com/spreadsheets/d/1ZRf1HAbR1lTPS17ttHRKDAaav5KpmeAGuxq7ND2r17g/edit#gid=0

## Limitations

In CockroachDB {{page.version.version}}, cluster virtualization has the following limitations:

- Cluster virtualization is supported only for cluster-to-cluster replication. General-purpose virtual clusters are not supported.
- A single physical cluster can have a maximum one system interface and one virtual cluster.

## Tasks to move to separate topics

### Enable cluster virtualization

### Connect to the system interface

When cluster virtualization is enabled, you specify whether to connect to the system interface or a virtual cluster as part of a SQL client's connection string. To connect to the system interface, set the `ccluster` option to `system` by adding the following to your connection string:

{% include_cached copy-clipboard.html %}
~~~
options=-ccluster=system
~~~

For example:

{% include_cached copy-clipboard.html %}
~~~
cockroach sql --certs-dir=certs --url 'postgresql://root@localhost:9001/defaultdb?options=-ccluster=system&sslmode=verify-full'
~~~

### Grant access to the system interface

1. How?

### Manage virtual clusters

#### Create a virtual cluster

#### Connect to a virtual cluster

When cluster virtualization is enabled, you specify whether to connect to the system interface or a virtual cluster as part of a SQL client's connection string. To connect to a virtual cluster, set the `ccluster` option to the virtual cluster's name by adding the following to your connection string:

{% include_cached copy-clipboard.html %}
~~~
options=-ccluster={virtual_cluster_name}
~~~

Replace `{virtual_cluster_name}` with the name of the virtual cluster.

For example, to connect to a virtual cluster named `finance`:

{% include_cached copy-clipboard.html %}
~~~
cockroach sql --certs-dir=certs --url 'postgresql://root@localhost:9001/defaultdb?options=-ccluster=finance&sslmode=verify-full'
~~~

#### Grant access to a virtual cluster

#### Delete a virtual cluster

### Upgrade a cluster with virtualization enabled

When cluster virtualization is enabled, you must upgrade the system interface separately before upgrading virtual clusters. The system interface can be at most one version ahead of virtual clusters. For example, a system interface on CockroachDB v23.2 can have virtual clusters on CockroachDB v23.1. This allows you to upgrade virtual clusters gradually, and allows you to roll back an upgrade of the system interface without impacting schemas or data in virtual clusters.

#### Upgrade the system interface

1. How?

#### Upgrade a virtual cluster

1. How?
