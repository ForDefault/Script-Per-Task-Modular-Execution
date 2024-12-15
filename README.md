# Script-Per-Task-Modular-Execution
EOF-Script Loop Method - when you want each item on a list to have a script run for it. 

Example: 
check
0.0.0.0:8080
172.25.0.4:8080
localhost:8080
172.25.0.1:8080

with a curl command

```
#!/bin/bash
set -a  # Export all variables to be available to any subshells created.
```

# Define the list of IPs and ports to check.
```
# Here, we are creating a file `ip_list.txt` with the IP:Port entries to process.
cat <<'EOF' > ip_list.txt
0.0.0.0:8080
172.25.0.4:8080
localhost:8080
172.25.0.1:8080
EOF
```

> Create or confirm the existence of a completed list file.
```
# This file will store processed items and their results (e.g., SUCCESS/FAILED).
touch ip_list.completed.txt
```
> Function to create a progressive backup of `ip_list.txt` - like checking off or marking through a list of items. 
```
# This ensures we keep a backup of the original list before any modifications.
backup_list() {
    i=1  # Initialize backup index.
    # Loop to check for an existing backup file with a numbered name.
    while [[ -f "$i.ip_list.txt" ]]; do
        i=$((i + 1))  # Increment the index until an unused file name is found.
    done
    # Copy the current `ip_list.txt` to a new backup file with the determined name.
    cp ip_list.txt "$i.ip_list.txt"
    echo "Backup created: $i.ip_list.txt"  # Notify the user about the backup.
}
backup_list  # Call the backup function.
```
> Outer loop: Continue processing while `ip_list.txt` is not empty.

```
    # Dynamically generate a script for checking connections for the first item in the list.

while [ -s ip_list.txt ]; do
    # Double-check if the list is empty and break if so (redundant, but safe).
    if [ ! -s ip_list.txt ]; then
        break
    fi
```
  
# This part is key to the Script-Per-Task Modular Execution methodology.
```
    cat <<'EOF' > check_connection.sh
#!/bin/bash

# Ensure the script runs in the directory where it was launched.
cd "$(dirname "$0")"

# Sort the file list alphabetically to ensure consistent processing order.
# The sorted list overwrites the original `ip_list.txt`.
sort ip_list.txt -o ip_list.txt

# Extract the first IP:Port from the sorted list using `head`.
ip_port=$(head -n 1 ip_list.txt)

# Check the connection to the IP:Port using curl with a timeout of 5 seconds.
if curl -s --connect-timeout 5 http://$ip_port > /dev/null; then
    # If the connection is successful, output SUCCESS and set the result variable.
    echo "SUCCESS!: $ip_port"
    result="SUCCESS"
else
    # If the connection fails, output FAILED and set the result variable.
    echo "FAILED: $ip_port"
    result="FAILED"
fi

# Remove the processed IP:Port from the list using `sed`.
sed -i "1d" ip_list.txt

# Append the processed IP:Port and its result to the completed list.
echo "$ip_port - $result" >> ip_list.completed.txt
EOF
```
# The script runs for that single item and self cleans here
```
    # Make the dynamically generated script executable.
    chmod +x check_connection.sh

    # Execute the dynamically generated script to process the first IP:Port.
    ./check_connection.sh

    # Remove the dynamically generated script after execution to keep the workspace clean.
    rm -f ./check_connection.sh
done
```
# Now for each item the script will generate and run, displaying the output in terminal. 
```
echo "All IPs and ports processed. Exiting."
```


## The script as a whole:

```
#!/bin/bash
set -a  # Export all variables to be available to any subshells created.

# Define the list of IPs and ports to check.
# Here, we are creating a file `ip_list.txt` with the IP:Port entries to process.
cat <<'EOF' > ip_list.txt
0.0.0.0:8080
172.25.0.4:8080
localhost:8080
172.25.0.1:8080
EOF

# Create or confirm the existence of a completed list file.
# This file will store processed items and their results (e.g., SUCCESS/FAILED).
touch ip_list.completed.txt

# Function to create a progressive backup of `ip_list.txt`.
# This ensures we keep a backup of the original list before any modifications.
backup_list() {
    i=1  # Initialize backup index.
    # Loop to check for an existing backup file with a numbered name.
    while [[ -f "$i.ip_list.txt" ]]; do
        i=$((i + 1))  # Increment the index until an unused file name is found.
    done
    # Copy the current `ip_list.txt` to a new backup file with the determined name.
    cp ip_list.txt "$i.ip_list.txt"
    echo "Backup created: $i.ip_list.txt"  # Notify the user about the backup.
}
backup_list  # Call the backup function.

# Outer loop: Continue processing while `ip_list.txt` is not empty.
while [ -s ip_list.txt ]; do
    # Double-check if the list is empty and break if so (redundant, but safe).
    if [ ! -s ip_list.txt ]; then
        break
    fi

    # Dynamically generate a script for checking connections for the first item in the list.
    # This part is key to the Script-Per-Task Modular Execution methodology.
    cat <<'EOF' > check_connection.sh
#!/bin/bash

# Ensure the script runs in the directory where it was launched.
cd "$(dirname "$0")"

# Sort the file list alphabetically to ensure consistent processing order.
# The sorted list overwrites the original `ip_list.txt`.
sort ip_list.txt -o ip_list.txt

# Extract the first IP:Port from the sorted list using `head`.
ip_port=$(head -n 1 ip_list.txt)

# Check the connection to the IP:Port using curl with a timeout of 5 seconds.
if curl -s --connect-timeout 5 http://$ip_port > /dev/null; then
    # If the connection is successful, output SUCCESS and set the result variable.
    echo "SUCCESS!: $ip_port"
    result="SUCCESS"
else
    # If the connection fails, output FAILED and set the result variable.
    echo "FAILED: $ip_port"
    result="FAILED"
fi

# Remove the processed IP:Port from the list using `sed`.
sed -i "1d" ip_list.txt

# Append the processed IP:Port and its result to the completed list.
echo "$ip_port - $result" >> ip_list.completed.txt
EOF

    # Make the dynamically generated script executable.
    chmod +x check_connection.sh

    # Execute the dynamically generated script to process the first IP:Port.
    ./check_connection.sh

    # Remove the dynamically generated script after execution to keep the workspace clean.
    rm -f ./check_connection.sh
done

# Once all IPs and ports in the list are processed, output a completion message.
```

# Script-Per-Task-Modular-Execution

## EOF-Script Loop Method

The **EOF-Script Loop Method** is a modular execution technique designed to dynamically create, execute, and clean up scripts for individual items in a list. This method is ideal for scenarios where each list item requires separate processing, such as connectivity tests, file transformations, or database queries.

---

## Methodology

1. **Define Input Data**:
   - A file (`ip_list.txt`) holds the list of items to process.
   - A secondary file (`ip_list.completed.txt`) tracks processed items and results.

2. **Create Backups**:
   - A backup function ensures the original list is preserved, allowing recovery if needed.

3. **Outer Loop for Iterative Execution**:
   - The loop iterates through the list (`ip_list.txt`) until all items are processed.

4. **Dynamic Script Generation**:
   - A temporary script is generated dynamically using **EOF markers**. This script executes a specific command (e.g., `curl`) for the current item.

5. **Processing Logic in the Script**:
   - The generated script:
     - Sorts the list for consistency.
     - Extracts the first item using `head`.
     - Executes the task (e.g., connectivity test with `curl`).
     - Removes the processed item from the list with `sed`.
     - Logs the result to the completed list.

6. **Execution and Cleanup**:
   - The generated script is executed, then deleted to keep the workspace clean.

7. **Completion Notification**:
   - After processing all items, the script outputs a completion message and exits.

---

## Advantages

- **Encapsulation**: Processes each item in isolation.
- **Modularity**: Easily adaptable for different tasks or commands.
- **Safety**: Backups and result logs ensure reliability and recovery.
- **Automation**: Automates repetitive tasks with minimal manual intervention.

---

## Use Cases

1. **Port Scanning**:
   - Test connectivity for a list of IPs/ports (e.g., with `curl` or `nmap`).

2. **File Processing**:
   - Apply transformations or validations to a list of files (e.g., image compression).

3. **Database Queries**:
   - Run specific SQL queries for multiple databases or tables.

4. **CI/CD Automation**:
   - Execute isolated steps for multiple environments.

---

## Generalization Framework

1. **Take User Inputs**:
   - Accept a command template (e.g., `curl`) and a list of items.

2. **Replace Task Logic**:
   - Substitute the logic inside the generated script with the desired task or command.

3. **Standardize File Handling**:
   - Use consistent naming for input, backup, and result files.

4. **Adapt Cleanup**:
   - Adjust how temporary scripts and files are handled.
