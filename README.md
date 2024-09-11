# Flock Automation Script

This script automates the detection and resolution of errors in your Flock node by updating and restarting the node when necessary.

## Features

- **Error Checking**: 
  - The script scans the log file for specific error messages (e.g., "Validation failed").
  - If errors are found, the script determines that the node is not functioning correctly.

- **Updating**: 
  - When an error is detected, the script updates the project by pulling the latest changes from the main repository using `git pull`.
  - This ensures that the node runs the most recent code.

- **Restarting the Node**: 
  - Upon detecting an error, the script stops the currently running node in the existing screen session by sending the `Ctrl + C` command.
  - The script then restarts the node using the specified start command.

- **Continuous Monitoring**: 
  - The script continuously repeats these steps at set intervals (e.g., every 60 seconds).
  - If no errors are found, it logs that the node is running fine and proceeds to the next check.

## Benefits

- **Automated Error Handling**: Keeps your node operational by automatically updating and restarting it in case of errors or required updates.
- **Reduced Manual Intervention**: No need for manual checks or restarts, allowing for smoother and more reliable node operation.

---

This script ensures your Flock node remains operational by handling errors automatically and keeping it updated with the latest code.


# Flock Node Auto Restart Script Guide

This guide provides instructions on how to automate the restart of your Flock node if it encounters errors, such as stopping due to an update.

## A - For Nodes That Are Already Encountering Errors

If your node is already encountering errors and stopping frequently, follow these steps:

### 1. Start a New Screen Session and Enable Logging

Create a new screen session and enable logging to capture the node's activity:

```
screen -dmS flock -L -Logfile /root/llm-loss-validator/flock.log
```
### 2. Run the Node
Send the command to start the node within the created screen session:

```
screen -S flock -X stuff "bash start.sh --hf_token hf_xxxxxxxxxxxxxxxx --flock_api_key api_xxxxxxxxxxxxx --task_id 10 --validation_args_file validation_config_cpu.json.example --auto_clean_cache False\n"
```
### 3. Exit the Screen
Exit the screen session by pressing Ctrl + A followed by D.

## B - For Nodes That Are Running and You Want to Enable Logging in the Existing Screen Session
If your node is running and you want to enable logging in the existing screen session:

## 1. Open the Existing Screen Session
Assume the screen session name is flock:

```
screen -a -r flock
```
## 2. Set the Log File and Start Logging
While in the screen session, follow these steps:

-Press Ctrl + A and then press : to enter command mode.

-Enter the following command:

```
logfile /root/llm-loss-validator/flock.log
```
-Start logging by pressing Ctrl + A and then H.


**Summary**
Choose either A or B depending on your needs or preference to handle logging first.



## Automating the Node Restart with a Script  (The script should be run inside a screen session.)
### 1. Create the Script
First, create a script named restart_node.sh:
```
logfile /root/llm-loss-validator/flock.log
```
### 2. Paste the Following Code

```
#!/bin/bash

# Specify the name of your existing screen session
SCREEN_NAME="flock"

# Add your start commands here
START_COMMAND="bash start.sh \
--hf_token hf_xxxxxx \
--flock_api_key api_xxxxxxxxx \
--task_id 10 \
--validation_args_file validation_config_cpu.json.example \
--auto_clean_cache False"

# Error check function
check_error() {
    # Look for error messages in your log file
    if tail -n 50 /root/llm-loss-validator/flock.log | grep -q "Validation failed"; then
        return 1  # Error found
    else
        return 0  # No error
    fi
}

# Update function
update_project() {
    cd /root/llm-loss-validator
    git pull origin main
    cd src
}

# Node restart function
restart_node() {
    # Terminate commands running in the current session
    screen -S "$SCREEN_NAME" -X stuff '^C'
    sleep 2
    # Run the command in the current session
    screen -S "$SCREEN_NAME" -X stuff "$START_COMMAND\n"
}

# Main loop
while true; do
    check_error
    if [ $? -ne 0 ]; then
        echo "Error detected. Updating and restarting the node..."
        update_project
        restart_node
    else
        echo "No error, the node is running fine."
    fi
    # Set how frequently the loop runs
    sleep 60  # Wait 60 seconds and then recheck
done

```
## 3. Give the Script Execute Permissions

```
chmod +x restart_node.sh
```

## 4. Run the Script

```
./restart_node.sh
```
Exit the screen session by pressing Ctrl + A followed by D.





