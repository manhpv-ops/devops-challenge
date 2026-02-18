# Problem 3 Report: High Disk Usage on NGINX VM

## 1. Troubleshooting Methodology
To identify the source of the 99% disk usage on the Ubuntu VM, I would execute the following steps:

1.  **Verify Disk Usage**: Run `df -h` to confirm which partition is full (likely root `/`).
2.  **Locate Large Files**: Run `du -ahx / | sort -rh | head -20` or use `ncdu /` to identify the directories consuming the most space.
3.  **Check for Deleted Files**: Run `lsof +L1` to check for "zombie" files—files that have been deleted but are still held open by a process (common with NGINX logs).

## 2. Scenario A: Excessive Log Growth (Most Likely)
**Cause**: NGINX access or error logs have grown indefinitely. This often happens due to high traffic volume, debug logging being left enabled, or a broken/missing `logrotate` configuration.
**Impact**:
*   System cannot write new logs.
*   If the root partition fills completely, NGINX may stop processing requests (500 errors) or the OS may crash/panic.
**Recovery**:
*   **Immediate**: Truncate the large log files to reclaim space immediately without breaking the file handle: `truncate -s 0 /var/log/nginx/access.log`.
*   **Long-term**: Ensure `logrotate` is configured correctly to rotate, compress, and discard old logs.

## 3. Scenario B: Deleted Files Held by Process
**Cause**: An administrator or script deleted a large log file (e.g., `rm access.log`) to free space, but did not signal NGINX to release the file handle. The space is not reclaimed by the OS until the process releases it.
**Impact**: `df` shows 99% usage, but `du` does not show large files matching that usage.
**Recovery**:
*   **Immediate**: Reload NGINX to flush logs and release file handles: `sudo systemctl reload nginx` or `nginx -s reload`.
*   **Prevention**: Never manually delete active log files. Use `truncate` or proper log rotation signals.

## 4. Scenario C: Proxy Buffering or Caching Accumulation
**Cause**: If NGINX is configured for proxy caching or buffering large upstream responses, the `proxy_temp_path` or cache directory (e.g., `/var/cache/nginx`) may have filled up.
**Impact**: NGINX cannot buffer responses from upstream services, leading to dropped connections or incomplete transfers.
**Recovery**:
*   **Immediate**: Clear the cache directory manually.
*   **Long-term**: Configure the `proxy_cache_path` directive with a `max_size` parameter (e.g., `max_size=10g`) to ensure the cache automatically evicts old entries before filling the disk.

## 5. Prevention Strategy
To prevent recurrence:
1.  **Monitoring**: Set up an alert (e.g., Prometheus Node Exporter) to trigger when disk usage exceeds 80%.
2.  **Log Rotation**: Verify `/etc/logrotate.d/nginx` exists and is running daily.
3.  **Partitioning**: Ideally, mount `/var/log` on a separate partition so that log growth does not crash the OS.
