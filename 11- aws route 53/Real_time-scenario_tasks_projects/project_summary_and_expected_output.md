
# Project Summary: Implementing Route 53 Routing Policies for High Availability and Traffic Management

## Objective

The goal of this project is to configure and test various Amazon Route 53 routing policies to manage traffic efficiently between two AWS regions (North Virginia and Mumbai). This ensures high availability, optimized performance, and controlled traffic distribution for a web application running on EC2 instances.

## Scope of the Project

### 1. Infrastructure Setup

*   Launch EC2 instances in two AWS regions (North Virginia & Mumbai).
*   Install & configure Nginx web server to serve a test webpage.
*   Create a custom AMI for easy instance replication.

### 2. Route 53 Configuration

*   **Failover Routing:** Direct traffic to the primary instance and switch to the secondary instance in case of failure.
*   **Latency-Based Routing:** Route users to the nearest instance based on latency.
*   **Weighted Routing:** Distribute traffic between instances based on assigned weights.

### 3. Health Checks & Testing

*   Configure Route 53 health checks to monitor instance availability.
*   Simulate failures and verify traffic redirection.
*   Validate latency-based and weighted routing configurations.

## Key Outcomes

*   Automatic failover ensures high availability in case of regional failures.
*   Optimized latency routing improves user experience by reducing response time.
*   Controlled traffic distribution using weighted routing for better load balancing.

By the end of this project, a fully functional multi-region DNS failover system will be in place, demonstrating how Route 53 can intelligently route traffic based on health, location, and weight configurations.

# Expected Output After Implementing Route 53 Routing Policies

Once the Route 53 routing policies are successfully implemented and tested, users should expect the following outcomes:

### 1. Failover Routing: High Availability

*   **Scenario:** If the primary (North Virginia) instance is healthy, traffic is routed to it. When the primary instance fails, traffic automatically shifts to the secondary (Mumbai) instance.
*   **Expected Behavior:**
    *   Initially, `http://failover.your_domain_name.com` loads content from the North Virginia instance.
    *   After stopping Nginx on the primary instance, requests should start reaching the Mumbai instance within a few seconds.

*   **Verification:**
    *   Refreshing the webpage should now display content from Mumbai.
    *   Once the primary instance is restored, traffic should return to North Virginia.

### 2. Latency-Based Routing: Optimized Performance

*   **Scenario:** Users are directed to the instance with the lowest latency based on their geographical location.
*   **Expected Behavior:**
    *   Users from the USA should be routed to the North Virginia instance.
    *   Users from India should be routed to the Mumbai instance.

*   **Verification:**
    *   Accessing `http://latency.your_domain_name.com` from different locations should return responses from the nearest instance.
    *   Testing with tools like `curl` from different AWS regions should confirm latency-based redirection.

### 3. Weighted Routing: Controlled Traffic Distribution

*   **Scenario:** Traffic is split between the two instances based on weight values assigned.
*   **Expected Behavior:**
    *   If the weight is set as US: 1 and India: 254, almost all traffic should go to the Mumbai instance.
    *   A small percentage of requests will still be served from North Virginia.

*   **Verification:**
    *   Repeatedly refreshing `http://weighted.your_domain_name.com` should mostly return the Mumbai instance’s content.
    *   Occasionally, the response will be served from North Virginia based on the weighted probability.

## Final Outcomes & Benefits

*   **High Availability:** No downtime due to automatic failover.
*   **Optimized Performance:** Users experience the fastest response times.
*   **Traffic Control:** Distribute users based on business logic.

By the end of the implementation, the system will be resilient, geographically optimized, and intelligently managed through Route 53.

