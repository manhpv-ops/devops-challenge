# Solution

## Usage

To run the solution in the current directory:

```bash
chmod +x problem1
./problem1 TSLA sell
```

## Installation

To make the script a system-wide CLI command:

1. Make the script executable:
   ```bash
   chmod +x problem1
   ```

2. Move the script to `/usr/local/bin`:
   ```bash
   sudo mv problem1 /usr/local/bin/problem1
   ```

3. Verify the installation:
   ```bash
   problem1 TSLA sell
   ```
> NOTE: Update `LOG_FILE` and `OUTPUT_FILE` based on your environment.