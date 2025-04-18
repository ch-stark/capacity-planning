# ACM Capacity-planning
Our goal is to helps to create Capacity recommendation for ACM and Capacity recommendation for ACM Observability only.
- If this a brown field set up where the customer has a ACM hub running with a few real clusters being managed under it, we could use [ACM Inspector](https://github.com/bjoydeep/acm-inspector) to extract some data out of the system to give a recommendation.
- If this is a green field set up where the customer has no ACM running at all, proceed to read below. 

## ACM Observability Sizing

### Inputs Needed
We need the following information to calculate the resources required to run Observability on top of ACM.

1. number_of_managed_clusters: say 100
1. number_of_master_node_in_hub_cluster: say 3
1. number_of_addn_worker_nodes: say 47 and we assume these are above the base number: 3
1. number_of_vcore_per_node: say 16
1. number_of_application_pods_per_node: say 20. This is for the application and not ACM/Openshift workload.
1. number_of_namespaces: say 20. These are namespaces that will be created by the customer outside what is created by ACM/Openshift workload.
1. number_of_container_per_pod:1. Do not change this as of now.
1. number_of_samples_per_hour:12. This means metric will be sent every 5 min.
1. number_of_hours_pv_retention_hrs:24. This means that data will retained in PV for 24 hrs which is standard. Do not reduce this number for production systems.
1. number_of_days_for_storage:365. This the amount of days of retention in the object store.


### Methodology of Calculation
1. Firstly we have to calculate how many time series do we need to persist. This is calculated in 2 steps.
    1. Calculated Number of time series for Base 3M+3W deployment for a Cluster (based on lab data)
    1. Calcluated Number of Time series for Additional Worker Nodes
1. Then from total number of time series, we infer
    1. Memory requirement (2 hours of this time series data is stored in memory)
    1. CPU Requirement (this is a little weak at the moment. We have not seen too much dependency on CPU yet so we have not focussed too much on it)
    1. Disk needed for PVs (volume of data stored is dictated by settings in MultiCluster Observability CR)
    1. Storage needed for Object store (volume of data stored is dictated by settings in MultiCluster Observability CR)

### Calculator
- Use this [pythonNotebook](/calculation/ObsSizingTemplate.ipynb).
- Substitute the values in the notebook under section:
`Critical Input Parameters to Size` with values described above.
- Then run the notebook. It will produce the recommendations.
## ACM Sizing
We need the following information to calculate the resources required to run ACM. Observability sizing is `not` included here.

### Inputs Needed

1. managed_clusters: say 500
1. nos_of_worker_nodes_per_cluster: say 40 
1. nos_of_pods_per_cluster: say 5000
1. nos_of_namespace: say 200
1. nos_of_apps_per_cluster_distributed_by_hub: say 10. This implies ACM App life cycle is being used.
1. nos_of_policies_per_cluster: say 30
1. nos_of_acm_hub: say 10. We will assume the clusters to be distributed evenly across the hubs for sizing.


### Methodology of calculation
Each managed cluster and ACM applications and policies created on it creates corresponding objects on the Kubernetes APIServer of the ACM hub. As the number of managed cluster and corresponding applications and policies grow, the load on the Kubernetes APIServer increases. There is a finite limit on how big this footprint on APIServer can be - this is mainly governed by the size of database backing the APIServer ie etcd.

1. Check if the specified load profile is in danger of crossing this limit. That will merit more than 1 ACM hub
1. Check if the specified load profile merits a highly performant etcd system backed ie backed by NVMe disks
1. If topology of Hub Server is 3M+3W, check for corresponding data samples to size/project.
1. If topology of Hub Server is 3M, check for corresponding data samples to size/project.

### Calculator
This is manual as of now. Please reach out the Engineering team for this.
