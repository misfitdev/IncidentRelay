# Baseline/current benchmark

The saved [report](results/REPORT.md) compares upstream `29e7f82` with optimized
application `c9cb20c`. The report and charts are generated from the saved
measurements, request timelines, and compressed resource logs. Runtime metadata
and validation results accompany the measurements.

_Note: telegram container was disabled in both runs._

## Run the comparison

Use an isolated snapshot directory and a new output path:

```sh
mkdir -p /tmp/ir-intake/baseline /tmp/ir-intake/current /tmp/ir-intake/harness/tools/benchmarks
git archive 29e7f82 | tar -x -C /tmp/ir-intake/baseline
git archive c9cb20c | tar -x -C /tmp/ir-intake/current
cp tools/benchmarks/capacity.py tools/benchmarks/benchmark.conf /tmp/ir-intake/harness/tools/benchmarks/
docker pull ghcr.io/roxy-wi/incidentrelay:2.5
docker pull postgres:16-alpine
.venv/bin/python tools/benchmarks/run_comparison.py --root /tmp/ir-intake \
  --output /tmp/ir-intake-raw --seconds 20 --rounds 1
uv run --no-project --with matplotlib python tools/benchmarks/plot_comparison.py \
  /tmp/ir-intake-raw
```

The host client uses the normal Python application dependencies. The runner pins
runtime image IDs, uses a fresh disposable PostgreSQL database for each version,
and checks resource allocations and running-container inventory. Only loopback
ports 18380 and 55435 are published. It never uses the normal deployment's data.
The load generator runs on the host with a raised file-descriptor limit.

The workload uses unique alerts/groups and full explain traces. After 125 warmup
events, each version receives 20-second probes at 100/sec, 400/sec, and 128-client
saturation. Every successful response must match a persisted alert, distinct
group, completed trace, and trace steps. CPU counters are captured around HTTP
work; memory and resource activity are sampled about once per second.
PostgreSQL data/WAL use tmpfs. Memory comparisons use the same request count.

`--include-telegram` additionally checks each version's real unused Telegram
worker, with no configured channels or external Telegram traffic. The published
charts show only the disabled-worker comparison. The idle busy loop is fixed,
but it was not a material driver of the intake improvement. Running the optimized
container without configured channels did not appear to affect intake performance.

## Rebuild the saved charts

```sh
uv run --no-project --with matplotlib python tools/benchmarks/plot_comparison.py \
  tools/benchmarks/results
```

The renderer accepts saved measurements or a fresh benchmark run directory and
writes the comparison charts and report alongside its inputs. Runtime metadata,
request timelines, and resource counters remain available for inspection.
