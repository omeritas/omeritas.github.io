---
title: "Kubernetes Reconciliation and Triggers: Edge vs. Level"
date: 2024-04-28 15:14:00 +/-TTTT
---

When creating a Kubernetes controller and operator you have to decide the trigger options and reconciliation logic: level-based or edge-based. 

## Level-based Trigger and Reconciliation 

Kubernetes leverages a combination of level-based triggering and level-based reconciliation. A level-based trigger is a mechanism that constantly monitors the current state of your cluster's (custom) resources, comparing it against the desired state. 

When the current state differs from the desired state, controllers will reconcile through level-based reconciliation and make necessary adjustments to align the current state with the desired state. This makes the application it manages more resilient to failures, since it ensures that the system remains in the desired state. 

Example: Prometheus Operator

1. User specifies their desired state of the monitoring setup using the CRDs (Prometheus, Alertmanager etc).
2. The operator continuously monitors the actual state of these resources and compares it to the desired state specified by this user. 
3. If this is not the case, the operator will intervene to bring it back to the desired state (this concept is called **reconciliation loop**).

It is often used when stability is crucial, even at the cost of increased resource usage. During errors and crashes the operator will recover the applications by reapplying the desired state (**self-healing**).

## Edge-based Trigger and Reconciliation

Kubernetes can also make use of edge-based triggers and reconciliation for certain scenarios. An edge-based trigger reacts to a specific change or event within the cluster. 

When the edge-based trigger detects an event, it can initiate a reconciliation loop for the affected (custom) resource where it attempts to make the current state match the desired state.  Instead of constantly monitoring the application, they intervene when something changes or an event occurs. The operator will remain idle until needed.

Example: Horizontal Pode Autoscaler Operator

1. User defines thresholds for specific metrics (ex. CPU usage) using the HPA configuration, setting when pods should scale up or down.
2. The HPA Operator monitors these metrics against the thresholds. When a metric crosses a defined limit, it triggers the HPA Operator to intervene.
3. The HPA Operator adjusts the number of pods to align with the current demand.
In this case the operator will only intervene (adjust the number of pods) during specific events or triggers (a metric crosses a defined limit). 

This approach is ideal for environments where performance and resource efficiency are priorities, especially if changes are infrequent but require rapid responses when they occur. Edge-based operators can respond quicker to changes, because it immediately responds to specific events and triggers, instead of reconciling the entire state.

## Hybrid Approach

The problem with edge-triggered systems, however, is that it is less resilient and can result in wrong outcomes. Network issues, errors and unexpected events can obscure the true state of the system. Here is an example of how it could go wrong in an edge-triggered system:
1. You have 2 pod replicas running.
2. You instruct that you want to add 3 more replicas (2+3 = 5 pods running).
3. An unexpected event occured, so it only added 1 replica instead of 3 (2+1 = 3 pods running).
4. You instruct that you want to delete 3 replicas (3-3 = 0 pods running).
5. You now have 0 pods running, instead of the desired 2.

Level-driven reconciliation would work better here. Then the controller would not treat each instruction as an isolated action. Instead, it constantly compares the current number of replica pods against your desired end state of 2 replicas. The ReplicaSet controller uses this hybrid approach. If you decide to create a fully edge-based operator, make sure you have implemented proper error handling and resync mechanisms.

If you want to read more about this, check out:
- https://dominik-tornow.medium.com/the-mechanics-of-kubernetes-ac8112eaa302
- https://hackernoon.com/level-triggering-and-reconciliation-in-kubernetes-1f17fe30333d
- https://www.oreilly.com/library/view/programming-kubernetes/9781492047094/
