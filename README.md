# Request to termination pods

## Steps to reproduce
1. Run
```
$ make install
```
2. Edit the `httpbin` deployment and add the following configuration
```
...
template:
    metadata:
      annotations:
        proxy.istio.io/config: |
          terminationDrainDuration: 300s
          drainDuration: 300s
          proxyMetadata:
            EXIT_ON_ZERO_ACTIVE_CONNECTIONS: 'true'
    spec:
      ...
      terminationGracePeriodSeconds: 300
      ...
...
```
3. Exec into the `httpbin` pod's `istio-proxy` container and run `netstat`
```
$ watch "netstat -tnp | grep 15006"
```
4. Tail the `httpbin` istio-proxy logs
```
$ kubectl logs -f <POD> -c istio-proxy
```
5. From the sleep pod make a few requests
```
$ kubectl exec -it -n <SLEEP_NS> <SLEEP_POD> -c sleep -- curl -v http://httpbin.mesh.global/delay/2 --resolve "httpbin.mesh.global:80:240.0.10.1"
```
6. Make a few requests so that you have a more than one connections established
7. Scale down the `httpbin` deployment
8. You would see connections still intact and proxy logs would show `There are still X active connections`