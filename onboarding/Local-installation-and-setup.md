# Onboarding: Setting Up Your Mac For Local Development
### ⚠️ PLEASE USE [[Onboarding: Setting Up Your Mac For VM Development]] UNLESS YOU ARE COMFORTABLE WITH DIGGING YOURSELF OUT OF HOLES. THIS IS NOT A FULLY SUPPORTED APPROACH, SO SELF-SUFFICIENCY IS NEEDED!

## 0. Assumptions
- You have an M1+ mac. If you have an Intel chip this is utterly not worth it
- You are comfortable with macOS and also CLI tools, odd errors, and willing to debug things and tweak and share
- You have completed Step 1 of [[Onboarding: Setting Up Your Mac For VM Development]] (ie: the `curl`-based install script)

## 1. Trigger Local-Development Specific Script

🛑 👇🏽 🛑 👇🏽 🛑 👇🏽 🛑 👇🏽 🛑 👇🏽 🛑 👇🏽

**Check with your manager and ensure that you've been added to the appropriate Google Cloud projects before continuing (see [For Managers](https://github.com/binti-family/family/wiki/Onboarding:-Setting-Up-Your-Mac-For-VM-Development#for-managers) on the VM development setup page)**

Without this step you won't be able to download development dumps, and the below install script will fail

🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽

- Ensure you grab a new terminal session after running the above script
- Run the following in your terminal and do what is asked of you:
```bash
cd $HOME/family && ./scripts/b/lib/setup_mac_for_local_development.sh
```

## 2. Post-Setup Configuration

After the script completes, you may need to configure a few additional items:

### 2.1. GitHub Access Token

Some scripts (like `b unreleased-prs` and `b github-stats`) require a GitHub Personal Access Token:

1. Create a token at: https://github.com/settings/tokens/new
   - Name it (e.g., "Binti Family Development")
   - Select scopes: `repo` (full control of private repositories)
   - Copy the token immediately (you won't see it again)

2. Add it to your `.env` file:
```bash
echo "GITHUB_ACCESS_TOKEN=your_token_here" >> $HOME/family/.env
```

3. If you're using direnv, reload your environment:
```bash
cd $HOME/family
direnv allow  # or just cd . to reload
```

**Note:** The codebase also accepts `GH_TOKEN` as an alias for `GITHUB_ACCESS_TOKEN`.

### 2.2. Environment Variables in Shell

If you want environment variables from `.env` to be available in your shell (not just in Rails), you'll need direnv configured:

1. Verify direnv is configured (should be done by setup script):
```bash
cat ~/.config/direnv/direnv.toml
```

You should see:
```toml
[global]
load_dotenv = true
[whitelist]
exact = [ '/Users/YOUR_USERNAME/family/.env' ]
```

2. Create a minimal `.envrc` file in the project root (if it doesn't exist):
```bash
cd $HOME/family
echo "# direnv: automatically loads .env via load_dotenv = true" > .envrc
direnv allow
```

**Note:** The `.envrc` file is already in `.gitignore`, so it won't be committed. With `load_dotenv = true` in your direnv config, direnv will automatically load `.env` files when you `cd` into the directory.

## 3. Troubleshooting Common Issues

### 3.1. PostgreSQL Connection Issues (IPv6 vs IPv4 on macOS)

**Symptoms:**
- Rails fails to start with `ActiveRecord::ConnectionNotEstablished`
- Connection attempts to `::1` (IPv6) instead of `127.0.0.1` (IPv4)
- Error: `connection to server at "127.0.0.1", port 5432 failed`

**Solution:**
macOS may resolve `localhost` to IPv6 (`::1`) instead of IPv4 (`127.0.0.1`). Force IPv4 by adding to your `.env` file:

```bash
echo "PGHOST=127.0.0.1" >> $HOME/family/.env
echo "PGUSER=$(whoami)" >> $HOME/family/.env
```

Verify your PostgreSQL container is running:
```bash
docker ps | grep postgres
```

If it's not running:
```bash
cd $HOME/family && ./b docker-compose up -d
```

### 3.2. Redis Connection Issues

**Symptoms:**
- Sidekiq fails to start
- `Connection reset by peer (redis://localhost:6379)`
- Redis connection errors during Rails startup

**Solution:**
Similar to PostgreSQL, force IPv4 for Redis. The codebase should already use `127.0.0.1` in `RedisConfig`, but if you see issues:

1. Verify Redis container is running:
```bash
docker ps | grep redis
```

2. Check that `config/configs/redis_config.rb` uses `redis://127.0.0.1` (not `localhost`)

3. If you see SSL-related errors, ensure your local Redis doesn't have SSL enabled (the codebase should handle this conditionally)

### 3.3. Database Snapshot Loading Issues

**Symptoms:**
- `b db-load-snapshot dev` fails with "database is being accessed by other users"
- SSH connection errors when connecting to GCP VMs

**Solution:**

1. **Terminate blocking database connections:**
```bash
psql -h 127.0.0.1 -U $USER -d postgres -c "SELECT pid, usename, application_name, state FROM pg_stat_activity WHERE datname = 'binti_family_development' AND pid != pg_backend_pid();"
```

If you see active connections, terminate them by replacing `PID` with the actual pid from the query above:
```bash
psql -h 127.0.0.1 -U $USER -d postgres -c "SELECT pg_terminate_backend(PID);"
```

2. **Fix SSH host key issues for GCP VMs:**
If you see "REMOTE HOST IDENTIFICATION HAS CHANGED" errors:
```bash
# Remove the offending key (backup is created automatically)
# Replace LINE_NUMBER with the line number mentioned in the error message
sed -i.bak 'LINE_NUMBERd' ~/.ssh/google_compute_known_hosts
```

### 3.4. Environment Variables Not Loading

**Symptoms:**
- `printenv` doesn't show variables from `.env`
- Scripts fail with "Environment variable not found"

**Solution:**

1. **For Rails:** Variables in `.env` are automatically loaded by `dotenv-rails` when Rails starts. No additional setup needed.

2. **For Shell:** If you want variables available in your shell:
   - Ensure direnv is installed and hooked into your shell
   - Create `.envrc` file (see section 2.2)
   - Run `direnv allow` in the project directory
   - Variables will load when you `cd` into the directory

3. **Verify direnv is working:**
```bash
cd $HOME/family
direnv status
printenv | grep GITHUB_ACCESS_TOKEN  # or whatever variable you're checking
```

### 3.5. Duplicate Entries in .zshrc

**Symptoms:**
- Multiple "BINTI HELPERS" sections in `~/.zshrc`
- Duplicate environment variable exports

**Solution:**
The setup script may append to `.zshrc` multiple times. Clean it up by:
1. Opening `~/.zshrc` in your editor
2. Removing duplicate sections (keep the most recent one with valid values)
3. Reload your shell: `source ~/.zshrc` or open a new terminal

## 4. The End!
The script tells you to grab a new terminal, and run `cd $HOME/family` and then `overmind start`. That's good advice!

You'll also want to setup your code editor, etc etc. [[Onboarding: Setting Up Your Mac For VM Development]] has some helpful tips and whatnot, but you should be basically good to go!

---
As of October 2025, this document supersedes the previous doc in the Page History
