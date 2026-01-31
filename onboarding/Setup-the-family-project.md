# Setup the Binti family project

## Installation Options

There are two primary ways to set up the `family` project for development. You will be using both, and for different reasons. Below is a discussion of the PROS/CONS and why you would use one or the other:

### Local Installation

Local installation runs the application and all services directly on your development machine. This approach offers:

- **Faster iteration**: No network latency when accessing services
- **Full control**: Direct access to all services and databases
- **Offline development**: Can work without network connectivity
- **Easier debugging**: Direct access to logs and processes
- **Resource efficiency**: No need to maintain a separate VM
- **Development data**: Works with development data snapshots, keeping production data off your local file system

**Best for**: Developers who want the fastest development experience and have the necessary tools installed locally (Docker, PostgreSQL, Redis, etc.).

### VM Installation

VM installation runs the application in a virtualized environment that closely mirrors production. This approach offers:

- **Production-like environment**: Matches production infrastructure more closely
- **Isolation**: Keeps development environment separate from your local machine
- **Consistency**: Same environment across team members
- **Easier onboarding**: Less local setup required
- **Resource isolation**: Development work doesn't impact your local machine's resources
- **Production data access**: Can load and work with production data snapshots for troubleshooting production-specific issues
- **Data security**: Keeps production data off local file systems while still allowing access for debugging

**Best for**: Developers who prefer a containerized approach, want production parity, need to troubleshoot production-specific issues, or have limited local resources.

----

# Installation Instructions

## Pre-requisites (Done by Binti DevOps)
1. Add new hire to dev environment via IAM
2. Add them to all other environments via Pulumi.
3. set up cursor account
4. set up google account and VM access

## Assumptions
- You have an M1+ mac. If you have an Intel chip this is utterly not worth it
- You are comfortable with macOS and also CLI tools, odd errors, and willing to debug things and tweak and share

## Latest Video
📽️ [Recording going through both approaches can be found](https://drive.google.com/file/d/1VbvV7goGmFuprGTM0LQiTTRRBCKDMoIQ/view)

## Set up (Done by New Employee)
### 1. Run the Developer Setup Script

- On your laptop's terminal, run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/binti-family/developer_setup/HEAD/setup_mac.sh)"
```

- Follow the prompts, sign into GitHub, enter passwords, etc etc
- When the script is complete you will be given two options of subsequent scripts to run

### 2. Run VM-Development Specific Script

- Ensure you grab a new terminal session after running the above script
- Run the following in your terminal:

```bash
cd $HOME/family && ./scripts/b/lib/setup_mac_for_vm_development.sh
```

### 3. Do Google-Specific Preparation

Before we can create a VM for you, we need you to authenticate and connect to a machine within our Google infrastructure. These steps walk you through that.

🛑 👇🏽 🛑 👇🏽 🛑 👇🏽 🛑 👇🏽 🛑 👇🏽 🛑 👇🏽

**Check with your manager and ensure that you've been added to the appropriate Google Cloud projects before continuing (see [For Managers](#for-managers) below)**

Without this step you won't be able to access the below VM, which is what triggers the SSH key generation and user awareness we need later.

🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽 🛑 ☝🏽

- Run this in your terminal:

```bash
gcloud auth login
```

(This will open up a browser window and ask you to grant permissions)

- Then run this exact command in your terminal. `dev-isaac-roh` is an arbitrarily chosen pre-existing VM that ensures that you are generally able to SSH into the VMs.

```bash
gcloud --project vaulted-fort-143519 compute ssh --zone=us-west1-b dev-isaac-roh
```

(This triggers gcloud's SSH key generation, propagation, and user awareness we need to create a VM for you)

- You should see a prompt ending in `@dev-isaac-roh:~$`. Once you see this: `exit`

If that doesn't work, try running it one more time with the `--troubleshoot` flag:

```bash
gcloud compute ssh dev-isaac-roh --project=vaulted-fort-143519 --zone=us-west1-b --troubleshoot
```

⚠️⚠️⚠️ **If you don't see the above prompt, let your manager know**

### 3. set GITHUB_ACCESS_TOKEN, WHODUNNIT_EMAIL, and other key env vars
After the script completes, you may need to configure a few additional items:

#### 3.1. GitHub Access Token

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

#### 3.2. WHODUNNIT_EMAIL 
1. Add it to your `.env` file:
```bash
echo "WHODUNNIT_EMAIL=your_email_here" >> $HOME/family/.env
```

#### 3.3. Environment Variables in Shell

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

### 4. Run `b gcloud-credentials` & `b google-oauth-credentials`
```bash
b gloud-credentials
```
- follow the prompts, login as your user

```bash
b google-oauth-credentials
```
- follow the prompts

### 5. Run local setup script
- Ensure you grab a new terminal session after running the above script
- Run the following in your terminal and do what is asked of you:
```bash
cd $HOME/family && ./scripts/b/lib/setup_mac_for_local_development.sh
```
### 6. run local server using overmind

```bash
overmind start
```

### 7. create the vm
Create the VM using the following command, using the username from your binti email address. For example, if your binti email is `firstname.lastname@binti.com`, use `firstname.lastname`.

```bash
b gcp-dev-vm create firstname.lastname
```

### 8. Setup VSCode - Connecting

- [Download VSCode](https://code.visualstudio.com/download), if needed
- Open VSCode
- Go to the `View` menu and choose `Terminal`
- Get back into the code repo

```bash
cd family
```

- Append your fancy new VM to your SSH config by running this in terminal:

```bash
b gcp-dev-vm ssh_config_append YOUR_NAME
```

**Note:** YOUR_NAME should be the name used to create the VM. The script will prefix the env name (ie `dev-`)

- Install [`Remote - SSH`](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension
- Follow along with the screenshots below to connect to your VM
- ![candidate_setup_01](https://github.com/binti-family/family/assets/7142/4ca4220f-1409-4b36-8425-e3c1604d5ceb)
- ![candidate_setup_02](https://github.com/binti-family/family/assets/7142/cace0bc8-2ae3-40ee-b870-683986c90404)
- ![setup_vscode_ssh_choose_your_vm](https://github.com/binti-family/family/assets/7142/9635c866-e091-4821-9d27-be3dca8f22a6)
- ![candidate_setup_04](https://github.com/binti-family/family/assets/7142/ac9f5247-9e9c-4c77-8cf5-aa956223550b)
- ![candidate_setup_05](https://github.com/binti-family/family/assets/7142/a4007c05-aef2-426a-ba9a-ab085beaf871)
- ![candidate_setup_06](https://github.com/binti-family/family/assets/7142/554c124e-8881-411a-8da0-db4d937b9afa)

### 9. Setup VSCode - Extensions and Settings

This is to get your VSCode editor in nice working order, with linting and whatnot. You should see the below in the bottom right corner of VSCode. Click `Install` and wait until those finish. Should take about 30 seconds or less

![setup_vscode_install_extensions](https://github.com/binti-family/family/assets/7142/e9c72503-365b-453f-a79e-70f0664a30e9)

### 10. Tokens

In order for our tooling to work, you should setup the various tokens for the various services we use

- In VSCode, go to the `View` menu and choose `Terminal` (if it's not already visible)
- Open your zsh profile

```bash
code ~/.zprofile
```

- Scroll to the bottom of the file and find something that looks like the below sample. Go through the process of creating new tokens and adding them in this file

```bash
# https://id.atlassian.com/manage-profile/security/api-tokens
export JIRA_AUTH_STRING="YOUR_EMAIL@binti.com:YOUR_TOKEN_FROM_ABOVE_HERE"
export WHODUNNIT_EMAIL="YOUR_EMAIL@binti.com"
# END BINTI HELPERS FROM `./b init`
```

- Save and close this file

### 11. Running the app

[Overmind](https://github.com/DarthSim/overmind) is what we use to manage our services.

You can simply run `overmind start` and then check `http://localhost:3000` once everything is up and running.

After that, you'll need to set up your own user in order to log in:

- edit `.env` and add `DISABLE_GOOGLE_OAUTH_ENFORCEMENT=true`
- in `rails console`, run:

```ruby
AgencyWorker.create!(
  agency: Agency.find_by(slug: Agency::BINTI_AGENCY_SLUG),
  role: :binti_admin,
  first_name: "Your First Name",
  last_name: "Your Last Name",
  email: "your.email@binti.com",
  password: "any.password",
  password_confirmation: "any.password",
  role_for_permissions:
    Role.from_slug(Permissions::DefaultRoles::Slugs::BINTI_ADMIN),
)
```

### 12. Set up Chrome Remote Desktop

You will use this to run things like Cypress and Selenium and/or if you just feel like clicking around. Hell, you could even download VSCode onto your VM and just do everything from Chrome Remote Desktop if you wanted to.

- Go to [Chrome Remote Desktop setup page](https://remotedesktop.google.com/headless)
- Click `Begin`
- It'll tell you to download and install Chrome Remote Desktop, but it's already installed for you. So click `Next`
- You'll see a screen with a button to "Authorize". Click `Authorize`
- Click the copy icon next to the code snippet for `Debian Linux`
- In your VM's terminal in VSCode, paste the code snippet you just copied in the above step
- It'll prompt you for a 6+ digit PIN. Enter something and remember it
- Once that command finishes, go to [Chrome Remote Desktop Access page](https://remotedesktop.google.com/access)
- You should see your machine listed `dev-YOUR_EMAIL-roh`. Click on it
- It'll ask for that PIN again... you do remember it right? (I always check "Remember my PIN on this device" at this step)
- You should see a blue desktop background image with a little white mouse on it. You did it!

---

## Logging in

Google SSO should work for logging into any environment with your Binti.com account.

Alternatively, you can log in with an email/password combo. When logging in on your VM, the email/password you should use is dependent on which environment's database you have loaded. If it's development, use your email/password you used for development. If you update your email/password in an environment, it can take a while for the dump and the environment to synchronize. It's generally a day behind.

When logging in with an email/password combo, you will need to disable the requirement to use Google SSO. This can be done by adding `DISABLE_GOOGLE_OAUTH_ENFORCEMENT=1` to your .env file in your local repo.

---

## Glossary

**"**_**Virtual Machine**_**"** aka **"**_**VM**_**"** aka **"**_**Host**_**"** - A virtual machine is a dedicated slice (in terms of resources like RAM, CPU, bandwidth, etc) of a much larger physical machine. An end user's (ie: you) experience of a VM is that it's not actually virtual. It presents itself as a full-fledged server. We use VMs for all kinds of things, but for the purposes of this document, each person interacting with our source code (engineers are the obvious group, but we also welcome others to have VMs as well) has their own dedicated VM that lives in Google's datacenter in Oregon, where our production machines also live.

You can think of your VM as the computer you use just to do coding work, and it is a remote machine you access through your physical laptop

- **How you access it** - Primarily through VSCode (as described above in section 5)
- **Example usage in context** - "_On your VM_"

**"**_**Container**_**"** - A container can be thought of as a super lightweight virtual machine. The difference between the two, for the purposes of this discussion, are that containers use as much of the host's resources and libraries as possible. This makes them much faster to boot up and makes it perfect for development purposes where we can get very close to recreating the actual architecture of our production environments.

One important distinction here is that the containers run on the host itself. In other words, whereas the VM (as described above) lives in Oregon and we connect to it from our laptops, the containers run on the VM directly.

### Where to find the Terminal application on your computer

- Invoke Spotlight (`⌘` + `[spacebar]`) and type `terminal` into the search bar
- Hit `[enter]` to open the Terminal application

---

## For Managers

- Add them to `dev` environment [via IAM](https://console.cloud.google.com/iam-admin/iam?referrer=search&project=vaulted-fort-143519)
- Add them to all other environments via Pulumi. [Example PR](https://github.com/binti-family/infrastructure/pull/337)
  - Add their email to the `EMPLOYEE_EMAIL` enum found [here](https://github.com/binti-family/infrastructure/blob/main/pulumi/packages/core/src/constants/employee.ts).
  - Assign them to a team [here](https://github.com/binti-family/infrastructure/blob/main/pulumi/packages/core/src/iam/team.ts).
  - Create a PR.
  - Once merged, BuildKite will make the necessary changes!

---

As of January 2026, this document supersedes the previous docs.
