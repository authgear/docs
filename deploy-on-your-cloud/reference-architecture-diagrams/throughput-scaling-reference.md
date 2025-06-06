# Throughput Scaling Reference

Here is table for suggested compute requirements at various throughput for infrastructure resources planning.&#x20;

<table><thead><tr><th width="266">Throughput (HTTP Requests)</th><th>Recommeded k8s Nodes</th><th>Recommened Database</th></tr></thead><tbody><tr><td>50 RPS</td><td>3 x 2 vCPUs, 8 GB Ram node</td><td>2 vCPUs, 7.5 GB Ram</td></tr><tr><td>70 RPS</td><td>3 x 2 vCPUs, 8 GB Ram node</td><td>8 vCPUs, 30 GB Ram</td></tr><tr><td>100 RPS</td><td>6 x 2 vCPUs, 8 GB Ram node</td><td>8 vCPUs, 30 GB Ram</td></tr></tbody></table>

## Notes

* The provided HTTP RPS was tested with a mixed scenario that includes https calls for signup, login, refresh access tokens, etc.&#x20;
* Some user action such as sign up requires multiple HTTP requests.
