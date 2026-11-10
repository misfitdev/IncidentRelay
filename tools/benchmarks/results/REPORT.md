# Alert-processing benchmark

Baseline and current application.

_Note: telegram container was disabled in both runs._

![Comparison](comparison.png)

![CPU and memory timelines](resources.png)

| Measurement | Baseline | Current |
|---|---:|---:|
| Completed requests/sec | 55.71 | 186.35 |
| CPU ms/completed request | 62.04 | 20.26 |
| Peak working-set MiB | 1,182.11 | 1,188.23 |
| p95 latency at 100/sec (ms) | 8,467.79 | 15.21 |
| Backlog after overload offer | 6,792.00 | 3,438.00 |
| Drain time (seconds) | 122.23 | 14.53 |
| Successful completions/sec during drain | 55.57 | 236.60 |

## Method

Upstream 29e7f82 and optimized c9cb20c use the same runtime images, four Gunicorn workers with four threads each, and a fresh PostgreSQL database. Limits: web three CPUs, PostgreSQL two CPUs, in a four-CPU Docker VM. The load generator runs on the host.
Each run receives 125 warmup events, then 20-second probes at 100/sec, 400/sec, and 128-client saturation. Saturation starts after the same 10,125 completed events. The overload offer contains 8,000 unique requests.
Every successful response is matched to persisted alerts, distinct groups, completed full traces and trace steps. CPU includes app and PostgreSQL. Memory is working-set usage and includes tmpfs database storage; memory is compared at the same request count.

Measurements and request timelines are in measurements.json; cumulative resource samples are in the two compressed resource logs. Validation results are in validation.json.

The Telegram worker's idle busy loop is also fixed. It was not a material driver of the measured intake improvement; the optimized container did not appear to affect intake performance when running without configured channels.
