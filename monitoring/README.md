# Monitoring namespace and all services

Several services are present in this namespaces:

1. Prometheus
1. Grafana
1. Pushgateway - a NodePort and not an ingress because Arduinos are pushing info to an IP and port
1. Smokeping
1. HS110 - 3 of them - one per TP-Link HS110 socket to monitor
