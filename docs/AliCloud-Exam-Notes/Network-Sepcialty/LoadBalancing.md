# Load Balancing

## 1. What is Load Balancing? 
- Distribute workload into multiple business processing units.
- Avoid overload of one single business processing node.
- Using multiple components with load balancing instead of a single component may increase reliability and availability through redundancy.
- Usually involves dedicated software or hardware.
![What is Load Balancing](images/whatisloadbalancing.png)

### Layer-4 Load balancing
- Distribute workload in OSI layer-4(TCP/IP).
- The backend unit is located in IP and port.
- No application support limitation.
![Layer-4 Load balancing](images/layer-4-loadbalancing.png)


### Layer-7 Load balancing
- Distribute workload in OSI layer-7(application).
- The backend unit is located in IP, port and application contents.
- Application is limited by the support application protocol(http, https...) of the load balancer.
![Layer-7 Load balancing](images/layer-7-loadbalacing.png)

## 2. Load Balancing Concept Introduction

### VIP 
- Virtual IP
- Act as unique IP entry point of the load balanced application.
- Normally located on the load balancer device.
- Multiple VIPs is supported on one load balancer for multiple application.
![VIP](images/VIP.png)

### None Session sticky
- All the request from outside will distribute into all backend servers equally.
- Disregard the source client IP (host) of each request.
![None Session sticky](images/NoSessionSticky.png)

### Session sticky
- All the request from one single source client IP(host) will be distributed to same backend server.
- Guarantee the session consistency from one source client.
![Session sticky](images/SessionSticky.png)

### Round Robin Mode
- All the request from outside will distributed into all backend servers on average.
![Round Robin Mode](images/RoundRobinMode.png)

### Weighted Round Robin Mode
- There will be weighted configured on each of the backend servers.
- Servers with higher weight value will be given more business requests.
- Normally servers with higher sepcification will be configured higher weight. 
![Weighted Round Robin Mode](images/WeightedRoundRobinMode.png)

### Least Connection Mode
- Servers will least connections at current moment will have higher priority to be distributed business requests.
![Least Connection Mode](images/LeastConnectionMode.png)

### Health Check
- Aimed to checked the working status of the backend servers.
- If one server is marked as failed status during the health check, it will not be distributed business request.
- It will recover after the failureb was fixed and the server will be marked as healthy again.
![Health Check](images/HealthCheck.png)

## 3. Common Adopted Load Balancing Solution

### Hardware based load balancing
- The load balancer entity is a hardware device.
- Normally dual machines are required for redundancy.
- Can provide stable and high performance load balancing.
- Normally the hardware load balancing device is expensive.
- Well-known brand: F5, Radware.
![Hardware based load balancing](images/HWLB.png)

### Software based load balancing
- The load balancer entity is software deployed on servers.
- Normally dual servers are required to configure HA mode of the load balancer for redundancy.
- Can provide various kinds of load balancing features, like MySQL support.
- Flexibly deployment on servers and servers are more cheaper than hardware load balancer.
- Well-known software LB solution: LVS, HAProxy, Nginx
![Software based load balancing](images/SWLB.png)


## 4. Alibab Cloud Load Balancing Solution

### SLB
- Server Load Balancer
- Software defined load balancer service from Alibaba Cloud is based on LVS and Tengine.
- Distribute the incoming traffic to multiple Elastic Compute Service (ECS) instances according to the configured forwarding rules.
- LVS for layer-4 load balancing and Tengine for layer-7 load balancing.
![SLB](images/SLB.png)

### SLB -  High Availability
- SLB support the distribute traffic to ECS instance across multiple Zones in one region.
- SLB itself can support to work in HA mode to have active and standby SLB instance in two zones.
- If the primary zone failed, the SLB service will switchover to the standby instance without affecting the service.
- The remaining ECS instances in remaining zone will also continue to process the business.
![SLB High Availability](images/SLBHighAvailability.png)

### SLB -  Disaster Recovery
- Combined with Cloud DNS service from Alibaba Cloud, SLB can achieve disaster recovery function.
- The business will be deployed across multiple regions with different Public IP addresses on SLB for each region.
- DNS will provide unique domain name as unique service entry point and make resolution to  multiple Public IP addresses.
- Once one whole region is failed, the DNS will make the public IP on the remained region as the resolution IP, thus achieving disaster recovery.
to process the business.
![SLB -  Disaster Recovery](images/SLBDisasterRecovery.png)

### SLB -  Health Check
- SLB support TCP and UDP health check for layer-4 LB and HTTP check for layer-7 LB.
- TCP health check is based on TCP 3-steps handshake procedure.
- UDP health check is based on UDP probing message.
- HTTP health check is based on http head request and response procedure.
![SLB -  Health Check](images/SLBHealthCheck.png)

### SLB - Security
- Since SLB will provide service on the internet, Alibaba can provide Anti-DDoS protection on SLB.
- By default, Anti-DDoS basis protection can protect SLB from DDoS attack under 5 Gbps.
![SLB - Security](images/SLBSecurity.png)