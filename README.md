# AzureLoganalyticsQueriesForAKS
###LogAnalytics queries for AKS Monitoring insight

> Get Node Count details for specified AKS cluster

```sh
KubeNodeInventory    | where ClusterName == 'Your ClusterName' | distinct  ClusterName, Computer, Status 
| summarize TotalCount = count(), ReadyCount = sumif(1, Status contains ('Ready'))                 by ClusterName   
| extend NotReadyCount = TotalCount - ReadyCount 
```

> Get CPU and Memory percentage details for specified AKS cluster

```sh
KubeNodeInventory    | where ClusterName == 'Your ClusterName' 
| distinct  ClusterName, Computer
| join hint.strategy=shuffle ( Perf |  where ObjectName == 'K8SNode' | where CounterName == 'cpuCapacityNanoCores' or CounterName == 'memoryCapacityBytes'        
| summarize LimitValue = max(CounterValue) by Computer, CounterName  | project Computer, CounterName = iif(CounterName == 'cpuCapacityNanoCores', 'cpu', 'memory'),               LimitValue    ) on Computer
| join kind=inner hint.strategy=shuffle (        Perf    
| where ObjectName == 'K8SNode'        | where CounterName == 'cpuUsageNanoCores' or CounterName == 'memoryRssBytes'       
| project Computer, CounterName = iif(CounterName == 'cpuUsageNanoCores', 'cpu', 'memory'), UsageValue = CounterValue ) on Computer, CounterName
| project ClusterName, Computer, CounterName, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize  Avg = avg(UsagePercent),  P50 = percentiles(UsagePercent, 50, 90, 95) by ClusterName, CounterName
```

> Get Pod Count details for specified AKS cluster

```sh
KubePodInventory       | where ClusterName == 'mdsp-advs-dev-aks'  | distinct ClusterName, Computer, PodUid, PodStatus| summarize TotalCount = count(),RunningCount = sumif(1, PodStatus =~ 'Running'), PendingCount = sumif(1, PodStatus =~ 'Pending'), FailedCount = sumif(1, PodStatus =~ 'Failed'),  SucceededCount = sumif(1, PodStatus =~ 'Succeeded')   by ClusterName 
```