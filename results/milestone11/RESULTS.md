# Milestone 11 - RPC and Distributed Shared State Results

## Test environment

Measurements were collected on the local Docker Compose deployment consisting of two worker containers (edge and core), one transparency gateway, and Kafka.

Run ID: `653f5533bb`

These results characterize the tested environment and workload, not a production-scale cluster.

## 1. RPC communication performance

| Measurement | Result |
|---|---:|
| Total concurrent system.ping RPC requests | 100 |
| Successful requests | 100/100 |
| Concurrency | 10 |
| Throughput | 239.85 calls/s |
| Mean latency | 40.14 ms |
| P50 latency | 40.17 ms |
| P95 latency | 53.03 ms |

All 100 RPC requests completed successfully. The measured P95 latency indicates that 95% of requests completed within approximately 53.03 ms.

## 2. Distributed state and synchronization

| Measurement | Result |
|---|---:|
| Versioned replicated writes | 30 |
| Replica consistency checks | 30/30 matched |
| Final read consistency | Passed |
| Mean write latency | 62.68 ms |
| P95 write latency | 77.55 ms |
| Write throughput | 10.88 writes/s |
| Direct replica read latency | 19.83 ms |
| Estimated synchronization overhead | 42.85 ms |
| Average protocol messages per write | 18 |
| Average reported message bytes per write | 4,305.93 bytes |

All 30 state writes completed successfully. The edge and core replicas reported matching versions in all 30 immediate consistency checks.

Synchronization overhead is estimated as mean replicated write latency minus direct replica read latency. It is indicative rather than a perfectly controlled paired measurement.

Message counts and bytes represent the application-level communication measurements reported by the instrumentation and do not represent total network traffic.

## 3. Concurrency and compare-and-swap testing

A concurrent compare-and-swap (CAS) race was performed to evaluate protection against conflicting updates.

| Measurement | Result |
|---|---:|
| Successful CAS operations | 1 |
| CAS conflicts | 1 |
| Race consistency | Passed |

One update was accepted while the competing update was rejected due to a version conflict. This demonstrates the expected stale-write protection under the tested race condition.

## 4. Tombstone consistency

The distributed deletion test verified that deletion information was maintained consistently across the replicas.

| Measurement | Result |
|---|---:|
| Tombstone consistency | Passed |
| Deleted tombstone version | 33 |

The test confirms that the deletion version was maintained consistently in the tested two-worker environment.

## 5. Distributed process RPC lifecycle

A separate RPC lifecycle trial documented in the earlier test results created a named process, resolved its name to a running resource, and terminated it through the gateway.

The recorded lifecycle was:

`created -> named -> located -> scheduled -> coordinated -> accepted`

Lookup and coordinated termination subsequently succeeded, with acknowledgements from both workers.

This lifecycle result is retained as a separate previously documented test and is not presented as a measurement from the latest benchmark run.

## 6. Consistency interpretation

The implemented gateway provides serialized writes and synchronous commits to every worker reachable during a write.

Gateway reads use the same serialization lock, select the highest replica version, reject equal-version conflicts, and repair older replicas.

Version-based compare-and-set prevents stale writers from overwriting newer state, while tombstones preserve deletion versions.

The latest test results demonstrate:

- Successful RPC communication.
- Successful replicated state writes.
- Matching edge and core replica versions.
- Correct CAS conflict handling.
- Consistent tombstone handling.

These results support the consistency behaviour of the implemented single-gateway, two-worker configuration.

## 7. Limitations

The implementation and measurements have the following limitations:

1. A single gateway serializes writes, limiting write concurrency.
2. Results characterize a two-worker Docker Compose environment and do not establish production-scale performance.
3. The tests do not establish consensus or linearizability across multiple gateways.
4. Availability during gateway failure has not been established.
5. A commit timeout may leave an uncertain transaction outcome; the transaction ID is returned in the error for diagnosis.
6. Write throughput is lower than RPC ping throughput because replicated writes require additional discovery, prepare, and commit exchanges.

## 8. Conclusion

Milestone 11 demonstrates successful RPC communication and distributed shared-state operations in the tested environment.

All 100 RPC requests succeeded, all 30 replica consistency checks matched, and the CAS and tombstone consistency tests passed.

The measurements provide evidence of the implemented communication, replication, synchronization, and concurrency mechanisms while identifying the limitations of the current architecture.

## 9. Raw measurement files

The machine-readable measurements are available in:

- `rpc_measurements.csv`
- `consistency_trials.csv`
- `summary.json`
