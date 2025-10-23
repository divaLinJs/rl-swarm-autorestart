# Gensyn RL-Swarm Auto-Restart Script

Automated deployment script for Gensyn RL-Swarm node with auto-restart, hang detection, and crash recovery.

## Features

- ✅ **Automatic dependency installation** (Python, Node.js, system packages)
- ✅ **Modal-login server management** with health checks
- ✅ **Intelligent watchdog** - detects crashes, fatal errors, and hangs
- ✅ **Automatic restart** on node failure
- ✅ **Log rotation** - keeps last 10 log files
- ✅ **Process cleanup** - prevents duplicate nodes and zombie processes
- ✅ **Inactivity detection** - kills hung nodes (30 min timeout)

## Installation

1. **Clone your rl-swarm repository:**
```bash
git clone  rl-swarm
cd rl-swarm
```

2. **Download the auto-restart script:**
```bash
wget https://raw.githubusercontent.com/divaLinJs/rl-swarm-autorestart/main/restart_rl_swarm.sh
chmod +x restart_rl_swarm.sh
```

Or manually:
```bash
curl -O https://raw.githubusercontent.com/divaLinJs/rl-swarm-autorestart/main/restart_rl_swarm.sh
chmod +x restart_rl_swarm.sh
```

3. **(Optional) Place your backup authentication files:**
```bash
mkdir -p modal-login/temp-data/
cp /path/to/userData.json modal-login/temp-data/
cp /path/to/userApiKey.json modal-login/temp-data/
```

4. **Run the script:**
```bash
source venv/bin/activate && ./restart_rl_swarm.sh
```

## First-Time Setup

If you don't have backup JSON files, the script will:

1. Start modal-login server
2. Open a localtunnel URL for login
3. Wait for you to complete authentication in browser
4. Automatically extract ORG_ID and continue

## Configuration

### Environment Variables
```bash
# Skip API key activation check (recommended)
REQUIRE_API_KEY_ACTIVATION=0 ./restart_rl_swarm.sh

# Custom inactivity timeout (default: 1800s = 30 min)
INACTIVITY_TIMEOUT=2700 ./restart_rl_swarm.sh  # 45 minutes

# Disable console log output
SWARM_LOG_TO_CONSOLE=0 ./restart_rl_swarm.sh

# Use custom Gensyn version
GENRL_TAG=0.1.12 ./restart_rl_swarm.sh

# Custom contract addresses
SWARM_CONTRACT=0x... PRG_CONTRACT=0x... ./restart_rl_swarm.sh

# Reset config to default
GENSYN_RESET_CONFIG=1 ./restart_rl_swarm.sh
```

## How It Works

### 1. Initialization
- Checks and installs system dependencies (jq, curl, python3-dev, build-essential)
- Kills any existing rl-swarm processes to prevent conflicts
- Starts modal-login server on port 3000
- Waits for server readiness (30s timeout)

### 2. Authentication
- Checks for existing `userData.json` and `userApiKey.json`
- If missing: opens localtunnel for browser login
- Extracts ORG_ID from userData
- Optionally waits for API key activation

### 3. Python Environment
- Installs/upgrades pip
- Installs gensyn-genrl, reasoning-gym, and hivemind
- Syncs configuration files

### 4. Node Execution
- Starts rl-swarm node
- Spawns watchdog process for monitoring
- Tails logs to console (optional)

### 5. Watchdog Monitoring

The watchdog continuously monitors for:

**Fatal Error Patterns:**
- `Resource temporarily unavailable`
- `EOFError: Ran out of input`

**Inactivity Detection:**
- Tracks log file size changes
- Kills node if no activity for 30 minutes (configurable)
- Prevents indefinite hangs

**Actions on Detection:**
- Kills hung/crashed process
- Returns to main loop
- Restarts node after 5 seconds

### 6. Auto-Restart Loop
- Infinite restart loop
- 5-second delay between restarts
- No crash limit (runs forever)

## Log Files
```
logs/
├── swarm_YYYYMMDD_HHMMSS.log  # Node logs (rotated, keeps last 10)
├── yarn_install.log            # modal-login yarn install
├── yarn_build.log              # modal-login build
├── yarn_start.log              # modal-login server
├── python_deps.log             # Python package installation
└── lt.log                      # Localtunnel output
```

## Troubleshooting

### Port 3000 Already in Use
```bash
# Kill process on port 3000
fuser -k 3000/tcp

# Or check what's using it
lsof -i :3000
```

### Multiple Nodes Running
```bash
# Check for duplicate processes
ps aux | grep swarm_launcher

# Kill all rl-swarm processes
pkill -f "rgym_exp.runner.swarm_launcher"
pkill -f "wandb-core"
```

### Modal-Login Won't Start
```bash
# Check logs
tail -100 logs/yarn_start.log

# Manually test
cd modal-login
yarn start
```

### Localhost Connection Issues

Check iptables rules allow localhost traffic:
```bash
# Check OUTPUT chain
iptables -L OUTPUT -n -v

# Allow localhost if blocked
iptables -I OUTPUT -d 127.0.0.0/8 -j ACCEPT
iptables -I INPUT -s 127.0.0.0/8 -j ACCEPT
```

### Node Keeps Crashing
```bash
# Check latest log
tail -100 logs/swarm_*.log | tail -100

# Check for memory issues
free -h
htop
```

## Testing Watchdog

### Test Fatal Error Detection
```bash
# Add fatal pattern to log
echo "EOFError: Ran out of input" >> logs/swarm_*.log

# Watchdog should kill process within 5 seconds
```

### Test Hang Detection
```bash
# Find node PID
ps aux | grep swarm_launcher

# Freeze process
kill -STOP <PID>

# Watchdog should kill after 30 minutes of inactivity
```

### Test Manual Restart
```bash
# Find node PID
ps aux | grep swarm_launcher

# Kill process
kill <PID>

# Should restart within 5 seconds
```

## System Requirements

- Ubuntu 22.04+ / Debian 11+
- Python 3.12+
- 32GB+ RAM recommended
- Internet connection for testnet

## File Structure
```
rl-swarm/
├── restart_rl_swarm.sh        # Main script
├── modal-login/                # Modal-login server
│   ├── temp-data/
│   │   ├── userData.json      # Your auth data (gitignored)
│   │   └── userApiKey.json    # Your API key (gitignored)
│   └── .env                    # Contract addresses
├── rgym_exp/                   # RL experiment code
├── configs/                    # Configuration files
├── logs/                       # Log files
└── swarm.pem                   # Node identity (auto-generated)
```

## Security Notes

- **Never commit** `userData.json`, `userApiKey.json`, or `swarm.pem` to git
- Add to `.gitignore`:
```
  modal-login/temp-data/
  *.pem
  logs/
  .env
```

## Contributing

Feel free to submit issues or pull requests to improve the script!

## Support

For issues with:
- **Script itself**: Open an issue in this repo
- **Gensyn platform**: Check [Gensyn docs](https://docs.gensyn.ai)
- **Modal-login**: Check modal-login repository

## License

MIT License - feel free to modify and distribute
