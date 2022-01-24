# Whoami

## Standalone NEG for whoami

```bash
# Create a firewall rule. Load balancers need to access cluster endpoints to perform health checks. This command creates a firewall rule to allow access
gcloud compute firewall-rules create whoami-fw-neg-rule \
   --network=ldf-infra-trs-vpc-001	 \
   --action=allow \
   --direction=ingress \
   --target-tags=gke-ldf-bridge-dev-gke-001-71e04038-node \
   --source-ranges=130.211.0.0/22,35.191.0.0/16 \
   --rules=tcp:80
```

```bash
# Create a global virtual IP address for the load balancer
gcloud compute addresses create whoami-server-global-static-ip \
    --ip-version=IPV4 \
    --global
```

```bash
# Create a health check. This is used by the load balancer to detect the liveness of individual endpoints within the NEG
gcloud compute health-checks create http whoami-http-basic-healthcheck \
    --use-serving-port
```

```bash
# Create a backend service that specifies that this is a global external HTTP(S) load balancer:
gcloud compute backend-services create whoami-bes \
    --protocol HTTP \
    --health-checks whoami-http-basic-healthcheck \
    --global
```

```bash
# create HTTP LB
# Create a URL map and target proxy for the load balancer. This example is very simple because the serve_hostname app used for this guide has a single endpoint and does not feature URLs.
gcloud compute url-maps create whoami-url-map \
    --default-service whoami-bes
gcloud compute target-http-proxies create whoami-http-lb-proxy \
    --url-map whoami-url-map
```

```bash
# Create a forwarding rule. This is what creates the load balancer.
# to set a specific external IP add this option : --address=HOSTNAME_SERVER_VIP \
gcloud compute forwarding-rules create whoami-http-forwarding-rule \
    --address=whoami-server-global-static-ip \
    --global \
    --target-http-proxy=whoami-http-lb-proxy \
    --ports=80
```

```bash
# Add backends to the load balancer (3 backend sur 3 zones différentes)
gcloud compute backend-services add-backend whoami-bes \
    --global \
    --network-endpoint-group=whoami-neg \
    --network-endpoint-group-zone=europe-west1-b \
    --balancing-mode RATE --max-rate-per-endpoint 5

gcloud compute backend-services add-backend whoami-bes \
    --global \
    --network-endpoint-group=whoami-neg \
    --network-endpoint-group-zone=europe-west1-c \
    --balancing-mode RATE --max-rate-per-endpoint 5

gcloud compute backend-services add-backend whoami-bes \
    --global \
    --network-endpoint-group=whoami-neg \
    --network-endpoint-group-zone=europe-west1-d \
    --balancing-mode RATE --max-rate-per-endpoint 5
```


### Checks 

```bash
# describe backend service
gcloud compute backend-services describe whoami-bes --global

gcloud compute backend-services get-health whoami-bes --global

gcloud compute addresses describe whoami-server-vip-test --global | grep "address:"
```

### Delete resources

```bash
# delete backends from backend service
gcloud compute backend-services remove-backend whoami-bes --network-endpoint-group=whoami-neg --global --network-endpoint-group-zone=europe-west1-b

gcloud compute backend-services remove-backend whoami-bes --network-endpoint-group=whoami-neg --global --network-endpoint-group-zone=europe-west1-c

gcloud compute backend-services remove-backend whoami-bes --network-endpoint-group=whoami-neg --global --network-endpoint-group-zone=europe-west1-d

# delete forwarding-rule
gcloud compute forwarding-rules delete whoamihttp-forwarding-rule --global 

# delete target http proxy
gcloud compute target-http-proxies delete whoami-http-lb-proxy

# delete url-map
gcloud compute url-maps delete whoami-url-map

# delete backend service
gcloud compute backend-services delete whoami-bes --global

# delete static IP 
gcloud compute addresses delete whoami-server-vip-test --global
```