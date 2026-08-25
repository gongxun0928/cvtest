# cvtest

Testing flow for Curvine, including CVbench, LTP, Xfstest and other customized cases.

## Status (2026-08-25)

LTP lane is live; xfstests and CVbench are planned (see "Roadmap").

## Layout

```
cvtest                     # CLI entrypoint (python3, stdlib only)
suites/ltp/runtest/        # curated LTP command files (installed into $LTP/runtest)
  cv-fs                    #   Curvine-curated syscall suite (initial version)
tools/ltp/install-ltp.sh   # pinned LTP installer (default /opt/ltp)
```

## Quick start

```bash
# 1. Install pinned LTP (default 20250930; the release must ship runltp —
#    upstream removed it in 20260529 in favor of kirk).
./tools/ltp/install-ltp.sh /opt/ltp

# 2. Ensure a Curvine cluster is mounted (see curvine repo:
#    build/dist + bin/curvine-{master,worker,fuse}.sh; fuse must be
#    started as root; probe with a touch+read on the mount).

# 3. Run suites.
./cvtest run --suite ltp:fs_perms_simple --mount /curvine-fuse --json out1.json
./cvtest run --suite ltp:cv-fs             --mount /curvine-fuse --json out2.json
```

Exit code: 0 = all tests passed AND runltp rc 0; 1 = failures; 2 = usage/env error.

## JSON contract

Each run emits one object whose fields match the curvine regression
Portal's `test_summary.json` `ltp_test` block (so
`curvine-tests/regression/tests/ltp_test.py` parsing semantics carry
over; `test_cases[].status` is `PASSED`/`FAILED`):

```json
{
  "tool": "ltp", "suite": "cv-fs", "status": "completed",
  "return_code": 0, "log_file": "...",
  "passed_count": 30, "failed_count": 0, "total_count": 30,
  "success_rate": 100.0,
  "test_cases": [{"name": "open01", "status": "PASSED"}, ...]
}
```

## Pinned tool versions

| Tool | Version | Why |
|------|---------|-----|
| LTP  | 20250930 | newest release verified green on Curvine that still ships `runltp`; 20260529+ removed runltp in favor of kirk |

## Roadmap

1. LTP: installer, cv-fs initial suite, cvtest run entry (done 2026-08-25)
2. curvine CI: `e2e-ltp.yml` workflow (label/nightly/workflow_dispatch; NOT a PR gate until privileged FUSE proves stable on hosted runners)
3. xfstests: pick runnable groups from Curvine's POSIX support matrix, add per-group command files
4. CVbench / fio performance lane on the same runner contract
5. kirk migration for LTP >= 20260529 (runltp removed upstream)
