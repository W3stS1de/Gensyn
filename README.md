# Gensyn RL-Swarm Node - VPS Ubuntu Installation Guide

Complete step-by-step guide for installing and running a Gensyn RL-Swarm node on Ubuntu VPS.

---

## System Requirements

### Minimum Specifications
- **OS:** Ubuntu 20.04 LTS or Ubuntu 22.04 LTS
- **RAM:** 16GB (32GB recommended)
- **CPU:** 4 cores (8 cores recommended)
- **Storage:** 50GB free space
- **Network:** Stable connection with 100+ Mbit/s

---

## Quick Installation

### One-Command Installation
```bash
sudo apt update && sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof build-essential gcc g++ && git clone https://github.com/gensyn-ai/rl-swarm.git && cd rl-swarm && python3 -m venv .venv && source .venv/bin/activate && sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml && screen -S gensyn && ./run_rl_swarm.sh
```

**When the script prompts you:**
1. Push to Hugging Face? → Type `N` and press Enter
2. Model name? → Type `Gensyn/Qwen2.5-0.5B-Instruct` and press Enter
3. Prediction Market? → Type `n` and press Enter

**Detach from screen:** Press `Ctrl+A`, then press `D`

---

### Setup Login Tunnel

Open a **NEW SSH session** to your VPS and run:
```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && sudo dpkg -i cloudflared-linux-amd64.deb && cloudflared tunnel --url http://localhost:3000
```

Copy the generated URL, open it in your browser, and complete the login.

---

## Step-by-Step Guide

### Step 1: Update System and Install Dependencies
```bash
sudo apt update && sudo apt upgrade -y
```
```bash
sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof nano unzip build-essential gcc g++
```

**Verify installations:**
```bash
python3 --version  
git --version
screen --version
```

---

### Step 2: Clone RL-Swarm Repository
```bash
cd ~
git clone https://github.com/gensyn-ai/rl-swarm.git
cd rl-swarm
```

---

### Step 3: Create Python Virtual Environment
```bash
python3 -m venv .venv
source .venv/bin/activate
```

The terminal command line should now display `(.venv)`

---

### Step 4: Configure Training Parameters

**Edit configuration to reduce RAM usage:**
```bash
sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml
```

**Verify the change:**
```bash
grep "num_train_samples" code_gen_exp/config/code-gen-swarm.yaml
```

Should show: `num_train_samples: 1`

**Why this matters:** Using `train_samples: 1` reduces RAM usage from 12-15GB to 8-10GB and significantly reduces crash probability.

---

### Step 5: Launch the Node

**Start in a screen session:**
```bash
screen -S gensyn
```

**Run the node:**
```bash
./run_rl_swarm.sh
```

**Answer the prompts:**
- `Would you like to push models to Hugging Face Hub? [y/N]` → `N`
- `Enter the name of the model` → `Gensyn/Qwen2.5-0.5B-Instruct`
- `Prediction Market? [y/N]` → `n`

**Wait for:**
```
Building server
Started server process: XXXXX
>> Failed to open http://localhost:3000. Please open it manually.
>> Waiting for modal userData.json to be created...
```

**Detach from screen:** Press `Ctrl+A`, then `D`

**Reattach later with:** `screen -r gensyn`

---

## Login Setup

### Step 1: Install Cloudflared Tunnel

**Open a NEW SSH connection to your VPS:**
```bash
ssh root@YOUR_VPS_IP
```

**Install cloudflared:**
```bash
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

**Verify installation:**
```bash
cloudflared --version
```

---

### Step 2: Create Tunnel
```bash
cloudflared tunnel --url http://localhost:3000
```

You will see output like:
```
Your quick Tunnel has been created! Visit it at:
https://random-words-1234.trycloudflare.com
```

**Copy this URL.**

**Important:** Keep this terminal open while completing login!

---

### Alternative Tunnel Options

**If cloudflared doesn't work, try:**

**Option A - localhost.run:**
```bash
ssh -o StrictHostKeyChecking=no -R 80:localhost:3000 nokey@localhost.run
```

**Option B - serveo:**
```bash
ssh -R 80:localhost:3000 serveo.net
```

---

### Step 3: Complete Login in Browser

1. Open the tunnel URL in your web browser
2. Click **"Sign in with Gmail"** (NOT "Connect with Google")
3. Enter your email address
4. Check your email for the verification code
5. Enter the code on the login page
6. Click Submit

**Wait for confirmation message in browser.**

---

### Step 4: Verify Login Success

**Check that files were created:**
```bash
ls -la ~/rl-swarm/userData.json
ls -la ~/rl-swarm/swarm.pem
```

**If both files exist, login was successful!**

You can now close the tunnel terminal (Ctrl+C).

**Check node logs:**
```bash
screen -r gensyn
```

You should see:
```
Connected to Gensyn Testnet
Hello [node_name] [Peer_ID]
Using Model: Gensyn/Qwen2.5-0.5B-Instruct
```

**Detach:** `Ctrl+A, D`

---

## Discord Role Setup

### Prerequisites
- Node running for 24+ hours continuously
- Peer ID registered on blockchain
- Same email used for login, dashboard, and Discord

---

### Step 1: Install Go
```bash
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.4.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
```

**Add to PATH:**
```bash
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

**Verify:**
```bash
go version
```

---

### Step 2: Install gswarm
```bash
go install github.com/Deep-Commit/gswarm/cmd/gswarm@latest
```

**Verify:**
```bash
gswarm --version
```

---

### Step 3: Create Telegram Bot

**Open Telegram and find @BotFather:**

1. Send `/newbot`
2. Choose a name for your bot (e.g., "My Gensyn Bot")
3. Choose a username (must end in "bot", e.g., "mygensyn_bot")
4. Copy and save the bot token provided

**Example token:** `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`

---

### Step 4: Get Your Telegram Chat ID

**Find @userinfobot in Telegram:**

1. Start a conversation
2. Send `/start`
3. Copy your Chat ID (numeric value)

**Example ID:** `987654321`

---

### Step 5: Get Your EOA Address

**Visit Gensyn Dashboard:**

1. Go to https://dashboard.gensyn.ai
2. Login with the same email you used for node login
3. Copy your EOA address (starts with 0x...)

---

### Step 6: Run gswarm
```bash
gswarm
```

**Enter when prompted:**
- Bot Token: Paste your bot token from BotFather
- Chat ID: Paste your Chat ID from userinfobot  
- EOA Address: Paste your EOA from dashboard

**If you see error "No peer IDs found":**
- Wait 1-2 hours (node needs time to register on-chain)
- Check logs: `grep "is already registered" ~/rl-swarm/logs/swarm_launcher.log`
- If you see your Peer ID registered, try gswarm again

---

### Step 7: Link Discord and Telegram

**In Discord:**
1. Go to Gensyn server, channel `#swarm-link`
2. Type: `/link-telegram`
3. Copy the verification code provided

**In Telegram:**
1. Open your bot (the one you created)
2. Send: `/verify YOUR_CODE_HERE`
3. Wait for confirmation

**Check Discord:**
- You should now have the "Gswarm" role
- Check your profile or server member list

---

## Troubleshooting

### Node crashes on startup
```bash
cd ~/rl-swarm && deactivate && rm -rf .venv && git stash && git pull && python3 -m venv .venv && source .venv/bin/activate && sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml && ./run_rl_swarm.sh
```

---

### Hydra or import errors
```bash
cd ~/rl-swarm && rm -rf /tmp/hivemind-* && git pull && sudo pkill -9 python3 && rm -rf .venv && python3 -m venv .venv && source .venv/bin/activate && sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml && ./run_rl_swarm.sh
```

---

### P2P connection failed
```bash
cd ~/rl-swarm && rm -rf /tmp/hivemind-* && sudo pkill -9 python3 && screen -S gensyn -X quit && screen -S gensyn && source .venv/bin/activate && ./run_rl_swarm.sh
```

---

### Update node to latest version
```bash
cd ~/rl-swarm && deactivate && rm -rf .venv && git stash && git pull && python3 -m venv .venv && source .venv/bin/activate && sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml && ./run_rl_swarm.sh
```

---

## Useful Commands

### Check node status
```bash
screen -r gensyn
```

Press `Ctrl+A, D` to detach

---

### Restart node
```bash
cd ~/rl-swarm && sudo pkill -9 python3 && screen -S gensyn -X quit && screen -S gensyn && source .venv/bin/activate && ./run_rl_swarm.sh
```

---

### Check port 3000
```bash
sudo lsof -i:3000
```

---

### View logs
```bash
tail -f ~/rl-swarm/logs/swarm_launcher.log
```

---

### Find your Peer ID
```bash
grep "Hello" ~/rl-swarm/logs/swarm_launcher.log | tail -1
```

---

### Backup important files
```bash
cp ~/rl-swarm/userData.json ~/userData_backup.json
cp ~/rl-swarm/swarm.pem ~/swarm_backup.pem
```

---

## FAQ

**Q: Do I need a GPU?**

A: No, CPU-only works fine. GPU is optional for faster training.

---

**Q: Minimum RAM requirement?**

A: 16GB minimum, 32GB recommended. Use `train_samples: 1` for 16GB systems.

---

**Q: How long to get Discord role?**

A: 24-48 hours of uptime required for Peer ID registration.

---

**Q: Node keeps crashing after 30 minutes?**

A: Check `train_samples: 1` is set in config. Also verify RAM usage with `htop`.

---

**Q: Can I use temporary email?**

A: No, temporary emails are blocked. Use Gmail or permanent email.

---

**Q: How to update node?**

A: Run:
```bash
cd ~/rl-swarm && deactivate && rm -rf .venv && git stash && git pull && python3 -m venv .venv && source .venv/bin/activate && sed -i -E 's/(num_train_samples:\s*)2/\1 1/' code_gen_exp/config/code-gen-swarm.yaml && ./run_rl_swarm.sh
```

---

## Official Links

- **Website:** https://www.gensyn.ai
- **Dashboard:** https://dashboard.gensyn.ai
- **GitHub:** https://github.com/gensyn-ai/rl-swarm
- **Discord:** https://discord.gg/gensyn
- **Docs:** https://docs.gensyn.ai

---

**Last Updated:** December 8, 2025  
**Guide Version:** 1.1  
**Compatible with:** RL-Swarm v0.7.0+
