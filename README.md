🖥️ Gensyn Screen Monitoring & Setup Guide

This guide explains how to monitor your Gensyn screen session to automatically detect issues and restart Python processes if they crash or WandB errors occur. It also shows how to start the main Gensyn session in a separate screen.

1️⃣ Open the Monitoring Screen

First, create a screen session for monitoring:
```
screen -S gencheck
```
2️⃣ Create & Edit the Monitoring Script

Open a new .sh file for the monitoring script:
```
nano gencheck.sh
```

Then paste the following script:
```
#!/bin/bash

# =========================
# Screen Monitor Script
# =========================
# This script monitors a single screen session (e.g., "gensyn") and automatically:
#   - Checks if the screen is running
#   - Detects if Python crashed or is stuck
#   - Detects WandB-related issues
#   - Attempts to restart or "wake up" the session if issues are found
# Logs are printed with timestamps for easy monitoring.
# -------------------------

screen_name="gensyn"  # Name of your screen session

while true; do
  if screen -list | grep -q "\.${screen_name}"; then
    tmpfile="/tmp/${screen_name}_check.log"
    screen -S "$screen_name" -X hardcopy -h "$tmpfile"

    last_lines=$(tail -n 10 "$tmpfile")

    # Check for WandB errors
    if echo "$last_lines" | grep -qi "wandb"; then
      echo "$(date '+%Y-%m-%d %H:%M:%S') ⚠️ $screen_name stopped (wandb error), restarting..."
      sleep 10
      screen -S "$screen_name" -X stuff $'\003'  # Send Ctrl+C
      sleep 1
      screen -S "$screen_name" -X stuff $'\003'
      sleep 1
      screen -S "$screen_name" -X stuff $'\033[A\n'  # Send Up arrow + Enter

    # Check for Python crash or shell fallback
    elif echo "$last_lines" | grep -Eq "(\.venv|Traceback)"; then
      echo "$(date '+%Y-%m-%d %H:%M:%S') ⚠️ $screen_name Python stopped or crashed, sending Up arrow + Enter..."
      screen -S "$screen_name" -X stuff $'\033[A\n'

    else
      echo "$(date '+%Y-%m-%d %H:%M:%S') ✅ $screen_name is running normally"
    fi

  else
    echo "$(date '+%Y-%m-%d %H:%M:%S') ❌ $screen_name screen is closed"
  fi

  sleep 300  # Check every 5 minutes
done
```

💾 Save and exit: Ctrl+X → Y → Enter

3️⃣ Run the Monitoring Script in Background

Detach from the screen while keeping the script running:
```
bash gencheck.sh
```

Detach the screen session:
```
Ctrl + A, D
```

This will leave the monitoring running in the background.

4️⃣ Open the Main Gensyn Screen

Start a new screen session for Gensyn:
```
screen -S gensyn
```
5️⃣ Start the Gensyn Process

Run the main script inside this screen:
```
printf "n\n\n" | bash run_rl_swarm.sh
```

This initializes the Gensyn process in the gensyn screen.

✅ Summary

**gencheck screen monitors your main gensyn session.
Automatically detects Python crashes or WandB issues and attempts to fix them.
gensyn screen runs the main Gensyn RL Swarm process.
Both screens can be detached and left running in the background.**
