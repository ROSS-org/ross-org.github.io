---
layout: post
title:  "Simulation Engine Metrics Descriptions"
author: Caitlin Ross
category: instrumentation
---

This page provides explanations on each of the simulation engine metrics collected in the ROSS instrumentation layer.

| Metric            | Description   |
| --------          | ------------- |
| nevent_processed  | Number of forward events processed. |
| nevent_abort      | Number of events aborted. |
| nevent_rb         | Number of reverse events processed. |
| rb_total          | Total number of rollbacks (primary + secondary). |
| rb_prim           | Total number of primary rollbacks (i.e., caused by receiving an event from the past). |
| rb_sec            | Total number of secondary rollbacks (i.e., caused by receiving an anti-message). |
| fc_attempts       | Number of fossil collection attempts. |
| pq_qsize          | Size of the priority queue (splay tree). This is the number of forward events to be processed at this time. |
| network_send      | Number of network sends. |
| network_recv      | Number of network recvs. |
| num_gvt           | Number of GVT computations. |
| event_ties        | Event ties detected on PEs. |
| efficiency        | Rollback efficiency: 1 - (events rolled back / net events). If this number is positive, more forward events are being processed than reverse events.  If negative, we're spending more time on rollback processing. |
| virtual_time_diff | Difference in a KP's local virtual clock and GVT. Typically positive (i.e., so KP is ahead of GVT), but can be negative if the KP has not processed any events since the last GVT. |

The following metrics are based on cycle counters to determine number of cycles spent in each type of processing.

| Metric            | Description   |
| --------          | ------------- |
| net_read_time     | Time spent polling network for received events |
| net_other_time    | Time spent performing other network tasks |
| gvt_time          | Time spent in GVT computation |
| fc_time           | Time spent in fossil collection |
| event_abort_time  | Time spent processing aborted events |
| event_proc_time   | Time spent in event processing |
| rb_time           | Time spent processing rollbacks |
| pq_time           | Time spent on priority queue operations |
| cancel_q_time     | Time spent processing events in the cancel queue |
| avl_time          | Time spent in AVL tree operations |
