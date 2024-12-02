---
title: Warehouses
slug: /en/cloud/reference/warehouses
keywords: [compute separation, cloud, architecture, compute-compute, warehouse, warehouses, hydra]
description: Compute-compute separation in ClickHouse Cloud
---

# Warehouses

(This feature is in Private Preview, please contact support for enabling this feature for your ClickHouse Cloud organization)

## What is Compute-Compute Separation?

Each ClickHouse Cloud service includes:
- A group of ClickHouse nodes (or replicas) - 2 nodes for a **Development** tier service and 3 nodes for a **Production** tier service
- An endpoint (or multiple endpoints created via ClickHouse Cloud UI console), which is a service URL that you use to connect to the service (for example, `https://dv2fzne24g.us-east-1.aws.clickhouse.cloud:8443`)
- An object storage folder where the service stores all the data and partially metadata:

<br />

<img src={require('./images/compute-compute-1.png').default}
    alt='NEEDS ALT'
    class='image'
    style={{width: '200px'}}
/>

<br />

_Fig. 1 - current service in ClickHouse Cloud_

Compute-compute separation allows users to create multiple compute node groups, each with its own endpoint, that are using the same object storage folder, and thus, with the same tables, views, etc.

Each compute node group will have its own endpoint so you can choose which set of replicas to use for your workloads. Some of your workloads may be satisfied with only one small-size replica, and others may require full high-availability (HA) and hundreds of gigs of memory. Compute-compute separation also allows you to separate read operations from write operations so they don't interfere with each other:

<br />

<img src={require('./images/compute-compute-2.png').default}
    alt='NEEDS ALT'
    class='image'
    style={{width: '500px'}}
/>

<br />

_Fig. 2 - compute  ClickHouse Cloud_

In this private preview program, you will have the ability to create extra services that share the same data with your existing services, or create a completely new setup with multiple services sharing the same data.

## What is a Warehouse?

In ClickHouse Cloud, a _warehouse_ is a set of services that share the same data.
Each warehouse has a primary service (this service was created first) and secondary service(s). For example, in the screenshot below you can see a warehouse "DWH Prod" with two services:
- Primary service "DWH Prod"
- Secondary service "DWH Prod Subservice"

<br />

<img src={require('./images/compute-compute-8.png').default}
    alt='NEEDS ALT'
    class='image'
    style={{width: '800px'}}
/>

<br />

_Fig. 3 - Warehouse example_

All services in a warehouse share the same:

- Region (for example, us-east1)
- Cloud service provider (AWS, GCP or Azure)
- ClickHouse database version

You can sort services by the warehouse that they belong to.

## Access controls

### Database credentials

Because all in a warehouse share the same set of tables, they also share access controls to those other services. This means that all database users that are created in Service 1 will also be able to use Service 2 with the same permissions (grants for tables, views, etc), and vice versa. Users will use another endpoint for each service but will use the same username and password. In other words, _users are shared across services that work with the same storage:_

<br />

<img src={require('./images/compute-compute-3.png').default}
    alt='NEEDS ALT'
    class='image'
    style={{width: '500px'}}
/>

<br />

_Fig. 4 - user Alice was created in Service 1, but she can use the same credentials to access all services that share same data_

### Network access control

It is often useful to restrict specific services from being used by other applications or ad-hoc users. This can be done by using network restrictions, similar to how it is configured currently for regular services (navigate to **Settings** in the service tab in the specific service in ClickHouse Cloud console).

You can apply IP filtering setting to each service separately, which means you can control which application can access which service. This allows you to restrict users from using specific services:

<br />

<img src={require('./images/compute-compute-4.png').default}
    alt='NEEDS ALT'
    class='image'
    style={{width: '400px'}}
/>

<br />

_Fig. 5 - Alice is restricted to access Service 2 because of the network settings_

### Read vs read-write

Sometimes it is useful to restrict write access to a specific service and allow writes only by a subset of services in a warehouse. This can be done when creating the second and nth services (the first service should always be read-write):

<br />

<img src={require('./images/compute-compute-5.png').default}
    alt='NEEDS ALT'
    class='image'
    style={{width: '400px'}}
/>

<br />

_Fig. 6 - Read-write and Read-only services in a warehouse_

## Scaling
Each service in a warehouse can be adjusted to your workloads in terms of:
- Number of nodes (replicas). Currently, the minimum number of nodes (replicas) is 2.
- Size of nodes (replicas)
- If the service should scale automatically
- If the service should be idled on inactivity (cannot be applied to the first service in the group - please see the **Limitations** section)

## Changes in behavior
Once compute-compute is enabled for a service (at least one secondary service was created), the `clusterAllReplicas()` function call with the `default` cluster name will utilize only replicas from the service where it was called. That means, if there are two services connected to the same dataset, and `clusterAllReplicas(default, system, processes)` is called from service 1, only processes running on service 1 will be shown. If needed, you can still call `clusterAllReplicas('all_groups.default', system, processes)` for example to reach all replicas.

## Limitations

Because this compute-compute separation is currently in private preview, there are some limitations to using this feature. Most of these limitations will be removed once the feature is released to GA (general availability):

1. **Primary service should always be up and should not be idled (limitation will be removed some time after GA).** During the private preview and some time after GA, the primary service (usually the existing service that you want to extend by adding other services) will be always up and will have the idling setting disabled. You will not be able to stop or idle the primary service if there is at least one secondary service. Once all secondary services are removed, you can stop or idle the original service again.

2. **Sometimes workloads cannot be isolated.** Though the goal is to give you an option to isolate database workloads from each other, there can be corner cases where one workload in one service will affect another service sharing the same data. These are quite rare situations that are mostly connected to OLTP-like workloads.

3. **All read-write services are doing background merge operations.** When inserting data to ClickHouse, the database at first inserts the data to some staging partitions, and then performs merges in the background. These merges can consume memory and CPU resources. When two read-write services share the same storage, they both are performing background operations. That means that there can be a situation where there is an `INSERT` query in Service 1, but the merge operation is completed by Service 2. Note that read-only services do not execute background merges, thus they don't spend their resources on this operation.

4. **Inserts in one read-write service can prevent another read-write service from idling if idling is enabled.** Because of the previous point, a second service perform background merge operations for the first service. These background operations can prevent the second service from going to sleep when idling. Once the background operations are finished, the service will be idled. Read-only services are not affected and will be idled without delay.

5. **CREATE/RENAME/DROP DATABASE queries could be blocked by idled/stopped services by default (limitation will be removed in GA).** These queries can hang. To bypass this, you  can run database management queries with `settings distributed_ddl_task_timeout=0` at the session or per query level. For example:

```sql
create database db_test_ddl_single_query_setting
settings distributed_ddl_task_timeout=0
```

## Pricing

Extra services created during the private preview are billed as usual. Compute prices are the same for all services in a warehouse (primary and secondary). Storage is billed only once - it is included in the first (original) service.

## Backups

- As all services in a single warehouse share the same storage, backups are made only on the primary (initial) service. By this, the data for all services in a warehouse is backed up.
- If you restore a backup from a primary service of a warehouse, it will be restored to a completely new service, not connected to the existing warehouse. You can then add more services to the new service immediately after the restore is finished.

## Using Warehouses

### Creating a Warehouse

To create a warehouse, you need to create a second service that will share the data with an existing service. This can be done by clicking the plus sign on any of the existing services:

<br />

<img src={require('./images/compute-compute-7.png').default}
    alt='NEEDS ALT'
    class='image'
    style={{width: '800px'}}
/>

<br />

_Fig. 7 - Click the plus sign to create a new service in a warehouse_

On the service creation screen, the original service will be selected in the dropdown as the source for the data of the new service. Once created, these two services will form a warehouse.

### Renaming a Warehouse

There are two ways to rename a warehouse:

- You can select "Sort by warehouse" on the services page in the top right corner, and then click the pencil icon near the warehouse name
- You can click the warehouse name on any of the services and rename the warehouse there

### Deleting a warehouse

Deleting a warehouse means deleting all the compute services and the data (tables, views, users, etc.). This action cannot be undone.
You can only delete a warehouse by deleting the first service created. To do this:

1. Delete all the services that were created in addition to the service that was created first;
2. Delete the first service (warning: all warehouse's data will be deleted on this step).