---
title: "Kubernetes Operators: Level-based vs Edge-based"
date: 2024-04-28 15:14:00 +/-TTTT
---

When creating a Kubernetes controller or operator you have to decide between two main strategies: level-based or edge-based. In this blog I will shortly explain the difference between the two.

![Level-based vs. Edge-based](https://prod-files-secure.s3.us-west-2.amazonaws.com/cab9acad-fdea-44a2-8213-362129e08ba5/75d0e87d-1eac-4608-8b92-6ebe89343173/Level-based_vs._Edge-based.png)

## Level-based operators

These operators constantly monitor the application it manages. They continually check the current state of an application against what it should be and make any necessary adjustments. This makes the application it manages more resilient to failures, since it ensures that the system remains in the desired state. 

Example of a level-based operator: Prometheus Operator

1. User specifies their desired state of the monitoring setup using the CRDs (Prometheus, Alertmanager etc).
2. The operator continuously monitors the actual state of these resources and compares it to the desired state specified by this user. 
3. If this is not the case, the operator will intervene to bring it back to the desired state (this concept is called **reconciliation loop**).

In this case the operator is constantly running and will always intervene to align the actual state to the desired state. 

This approach is optimal for applications requiring continuous monitoring to ensure a consistent state. It is well-suited for complex environments where stability is crucial, even at the cost of increased resource usage. Most operators adopt this level-based strategy due to its effectiveness in maintaining system integrity. During errors and crashes the operator will recover the applications by reapplying the desired state (**self-healing**).

## Edge-based operators
These operators respond to specific events or triggers. Instead of constantly monitoring the application, they intervene when something changes or an event occurs. Thus the operator will remain idle until needed.

Example of an edge-based operator: Horizontal Pode Autoscaler Operator

1. User defines thresholds for specific metrics (ex. CPU usage) using the HPA configuration, setting when pods should scale up or down.
2. The HPA Operator monitors these metrics against the thresholds. When a metric crosses a defined limit, it triggers the HPA Operator to intervene.
3. The HPA Operator adjusts the number of pods to align with the current demand.
In this case the operator will only intervene (adjust the number of pods) during specific events or triggers (a metric crosses a defined limit). 

This approach is ideal for environments where performance and resource efficiency are priorities, especially if changes are infrequent but require rapid responses when they occur. Edge-based operators can respond quicker to changes, because it immediately responds to specific events and triggers, instead of reconciling the entire state.
