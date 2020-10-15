Kubernetes Audit Kata
=====================

Imagine you work for a wildly successful company which has many products in the healthcare sector. Each product runs in its own Kubernetes cluster. Because of the regulatory demands in the healthcare sector it's important that there is an audit trail showing what is deployed and running at any given time. 

You are developing a new Audit microservice. It should record what is happening in a kubernetes cluster for auditing purposes. This exercise describes the first increment of functionality to be delivered and provides some sample data you can use.

When you start the Audit service, you configure it with 
- the hostname for the kubernetes server
- the name of the cluster this service should monitor
- the relevant namespace within that cluster

Every minute the service should query the Kubernetes API to find out a list of all the available resources. For each resource that meets certain criteria (detailed below), query the API again to find out details about that resource. Write a file for each resource containing these details. When all the resource details have been fetched, create an archive containing all the files and name it after the date and time. This file should be written to a particular location and permissions set to read only. There will be a new file each minute.

Of all the resources Kubernetes knows about, the only ones we want to save for audit purposes have the properties "namespaced": true and a "verb" called "list". Note: in the sample data given, only the "configmaps" resource meets those criteria.

## Details
Assume a kubernetes hostname "sample.k8s.hostname" and cluster name "sample_cluster" and namespace "sample_namespace".

This will fetch a list of all resources:

GET https://sample.k8s.hostname/k8s/clusters/sample_cluster/api/v1?timeout=32s

The response is formatted as json and an example is found in the file [sample_resource_list.json](sample_data/sample_resource_list.json).

That json contains information about resources. To then fetch further information about a particular resource, for example "configmaps", need to make a request like this:

GET https://sample.k8s.hostname/k8s/clusters/sample_cluster/apis/extensions/v1beta1/namespaces/sample_namespace/configmaps?limit=500

That request also needs to have the "Accept" header set, like this:

Accept: application/json

The Audit service doesn't need to parse this json, only store it in a file named after the resource. For the example data given, this is one file "configmaps.json".

The archive of all the resource json files should be a tar.gz file written to the relevant cluster folder and named after the datetime in ISO format, for example:

/audit/sample.k8s.hostname/sample_cluster/sample_namespace/2020-10-31T071055.audit.tar.gz

## Error handling

Whatever errors occur when requesting information from k8s the audit service should always write an archive file every minute, even if it is empty. It should also write a log file with information about any errors encountered. For example:

/audit/sample.k8s.hostname/sample_cluster/sample_namespace/2020-10-31T071055.audit.log





