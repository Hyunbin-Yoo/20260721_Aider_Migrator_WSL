# How to Build

## Install RHEL 10 on WSL2
- [Create a new Red Hat Developer Account](https://developers.redhat.com/register)
- [Get latest RHEL 10 WSL2 Image](https://access.redhat.com/downloads/content/479/ver=/rhel---10/10.0/x86_64/product-software)
- Install WSL2
  ```Powershell
  PS> wsl --install
  ```
- Create WSL2 Directory
  ```Powershell
  PS> New-Item -ItemType Directory -Path "C:\WSL\RHEL10"
  ```
- Import the downloaded image
  ```Powershell
  PS> wsl --import "RHEL10" "C:\WSL\RHEL10" "$env:USERPROFILE\Downloads\rhel-10.0-x86_64-wsl2.tar.gz" --version 2
  The operation completed successfully.
  ```
- Verify installation
  ```Powershell
    PS> wsl --list --verbose
    NAME      STATE           VERSION
  * Ubuntu    Stopped         2
    RHEL10    Stopped         2
  ```
- Change default to RHEL 10
  ```Powershell
  PS> wsl --set-default RHEL10
  PS> wsl --list --verbose
    NAME      STATE           VERSION
  * RHEL10    Stopped         2
    Ubuntu    Stopped         2
  ```

## Configure RHEL 10
- Start RHEL 10
  ```Powershell
  PS> wsl
  [root@Windows user]#
  ```
- Register RHEL Installation
  ```shell
  [root@Windows user]# subscription-manager register
  Registering to: subscription.rhsm.redhat.com:443/subscription
  Username:
  Password:
  The system has been registered with ID: (UUID)
  The registered system name is: Windows
  ```
- Upgrade System and Install Basic Apps
  ```shell
  [root@Windows user]# dnf update -y
  [root@Windows user]# dnf install -y git wget curl ncurses nano dnf-plugins-core
  ```
- Create non-root user account
  ```shell
  [root@Windows user]# useradd -m -G wheel ordinaryuser
  [root@Windows user]# passwd ordinaryuser
  New password:
  Retype new password:
  passwd: password updated successfully
  ```
- Change default user to non-Root
  ```shell
  [root@Windows user]# nano /etc/wsl.conf
  [root@Windows user]# cat /etc/wsl.conf
  [boot]
  systemd = true

  [user]
  default=ordinaryuser
  ```
- Exit and Restart to confirm new default user
  ```shell
  [root@Windows user]# exit
  PS> wsl --terminate RHEL10
  The operation completed successfully.
  PS> wsl
  [ordinaryuser@Windows user]$ whoami
  ordinaryuser
  ```

## Install Development Environment
- Download `uv`
  ```shell
  [ordinaryuser@Windows user]$ curl -sSfL https://astral.sh/uv/install.sh | sh
  downloading uv 0.11.29 x86_64-unknown-linux-gnu
  installing to /home/a/.local/bin
    uv
    uvx
  everything's installed!
  ```
- Reload Shell Profile
  ```shell
  [ordinaryuser@Windows user]$ source ~/.bashrc
  ```
- Install `aider` via `uv`
  ```shell
  [ordinaryuser@Windows user]$ uv tool install aider-chat
  Resolved 108 packages in 2.15s
  Prepared 108 packages in 13.25s
  Installed 108 packages in 270ms
  Installed 1 executable: aider
  ```
- Import VSCodium GPG key
  ```shell
  [ordinaryuser@Windows user]$ sudo rpmkeys --import https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/-/raw/master/pub.gpg
  ```
- Add VSCodium Repository
  ```shell
  [ordinaryuser@Windows user]$ sudo dnf config-manager addrepo https://download.vscodium.com/rpms/vscodium.repo
  ```
- Install VSCodium
  ```shell
  [ordinaryuser@Windows user]$ sudo dnf install -y codium
  Complete!
  ```

## Configure sync with GitHub
- Create Workspace
  ```shell
  [ordinaryuser@Windows user]$ mkdir ~/GitHub
  ```
- Set `.gitignore`s
  ```shell
  [ordinaryuser@Windows user]$ git config --global core.excludesfile ~/.gitignore_global
  [ordinaryuser@Windows user]$ echo ".aider*" >> ~/.gitignore_global
  ```
- Add GitHub CLI Repository
  ```shell
  [ordinaryuser@Windows user]$ sudo dnf config-manager addrepo https://cli.github.com/packages/rpm/gh-cli.repo
  Updating Subscription Management repositories.
  Adding repo from: https://cli.github.com/packages/rpm/gh-cli.repo
  ```
- Install GitHub CLI
  ```shell
  [ordinaryuser@Windows user]$ sudo dnf install -y gh
  Installed:
    gh-2.96.0-1.x86_64

  Complete!
  ```
- Log in to GitHub (Select SSH and create SSH key)
  ```shell
  [ordinaryuser@Windows user]$ gh auth login
  ```
- Add GitHub as trusted to prevent host verification prompts later on
  ```shell
  [ordinaryuser@Windows user]$ mkdir -p ~/.ssh && chmod 700 ~/.ssh
  [ordinaryuser@Windows user]$ ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
  [ordinaryuser@Windows user]$ chmod 600 ~/.ssh/known_hosts
  ```
- Add environment variables
  ```shell
  [ordinaryuser@Windows user]$ nano ~/.bashrc
  [ordinaryuser@Windows user]$ cat ~/.bashrc
    # .bashrc

  # Source global definitions
  if [ -f /etc/bashrc ]; then
    . /etc/bashrc
  fi

  # User specific environment
  if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]; then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
  fi
  export PATH

  # Uncomment the following line if you don't like systemctl's auto-paging feature:
  # export SYSTEMD_PAGER=

  # User specific aliases and functions
  if [ -d ~/.bashrc.d ]; then
    for rc in ~/.bashrc.d/*; do
      if [ -f "$rc" ]; then
        . "$rc"
      fi
    done
  fi
  unset rc

  # --- SSH Agent Persistence (Suppresses Passphrase Prompts inside Scripts) ---
  if ! pgrep -u "$USER" ssh-agent > /dev/null; then
    ssh-agent -s > ~/.ssh/ssh-agent.env
  fi
  if [ -f ~/.ssh/ssh-agent.env ]; then
    source ~/.ssh/ssh-agent.env > /dev/null
    ssh-add -l >/dev/null 2>&1 || ssh-add ~/.ssh/id_ed25519
  fi

  export AIDER_ANTHROPIC_API_KEY='xxxx'
  export AIDER_DEEPSEEK_API_KEY='xxxx'
  export AIDER_GOOGLE_API_KEY='xxxx'
  export AIDER_OPENAI_API_KEY='xxxx'
  export AIDER_QWEN_API_KEY='xxxx'
  export AIDER_XAI_API_KEY='xxxx'

  alias codium="DONT_PROMPT_WSL_INSTALL=1 codium"

  # --- Automated Parallel Aider Lifecycle Engine ---
  function aider() {
    local current_dir=${PWD##*/}
    local history_dir="../${current_dir}_Aider_History"
    local config_file="${history_dir}/.aider.conf.yml"

    if [ -f "$config_file" ]; then
      echo "--> Pulling latest remote history transaction logs..."
      (cd "$history_dir" && git pull --rebase --quiet origin main 2>/dev/null)

      # Launch the underlying Aider binary mapping the local anchor config
      command aider --config "$config_file" "$@"

      echo "--> Streaming local prompt history modifications to Cloud..."
      (
        cd "$history_dir"
        git add .
        # Check for changes to prevent empty commit tracking spam
        if ! git diff-index --quiet HEAD --; then
          git commit -m "sync: append transaction log via local runtime wrapper"
          git push origin main
        else
          echo "--> Cloud state is already up-to-date."
        fi
      )
    else
      # Fallback out of parallel tracking if working outside bootstrap space
      command aider "$@"
    fi
  }

  # --- Deterministic Project Bootstrap & Migration Engine ---
  function project-init() {
    if [ -z "$1" ]; then
      echo "Usage: project-init <project-name>"
      return 1
    fi

    local project_name=$1
    local history_dir="${project_name}_Aider_History"
    local base_dir="$HOME/GitHub"

    echo "--- Assessing Environment: ${project_name} ---"

    # 1. Enforce ~/GitHub Workspace Directory Constraint
    mkdir -p "$base_dir"
    cd "$base_dir" || return 1

    # 2. Prevent Overwriting Local State
    if [ -d "$project_name" ] || [ -d "$history_dir" ]; then
      echo "Error: Directory '$project_name' or '$history_dir' already exists locally."
      echo "Please remove them or cd into them directly."
      return 1
    fi

    # 3. Detect Cloud State via GitHub CLI
    echo "--> Querying GitHub for existing repositories..."
    local has_app=false
    local has_history=false

    # Suppress output, we only care about the exit code (0 = exists)
    if gh repo view "$project_name" &>/dev/null; then has_app=true; fi
    if gh repo view "$history_dir" &>/dev/null; then has_history=true; fi

    # 4. Handle Public App Repository (Clone vs Create)
    if [ "$has_app" = true ]; then
      echo "--> State: Migration/Sync. Downloading existing public repository..."
      gh repo clone "$project_name" "$project_name"
    else
      echo "--> State: New. Bootstrapping public repository..."
      mkdir "$project_name" && cd "$project_name" || return 1
      git init

      # Removed --push here; repository is empty and has no commits to push yet
      gh repo create "$project_name" --public --source=. --remote=origin
      cd "$base_dir" || return 1
    fi

    # 5. Handle Private History Repository (Clone vs Create)
    if [ "$has_history" = true ]; then
      echo "--> Downloading existing private history repository..."
      gh repo clone "$history_dir" "$history_dir"
    else
      echo "--> Bootstrapping new private history repository..."
      mkdir "$history_dir" && cd "$history_dir" || return 1
      git init

      # Inject Properties FIRST so a valid file asset exists to track
      cat <<EOF > .aider.conf.yml
  chat-history-file: ../${history_dir}/chat_history.md
  input-history-file: ../${history_dir}/input_history
  restore-chat-history: true
  EOF

      git add -f .aider.conf.yml   # <--- Add the -f flag here
      git commit -m "Initialize portable Aider profile"

      # Safe to push now because an initial commit provides a tracking ref (main)
      gh repo create "$history_dir" --private --source=. --remote=origin --push
      cd "$base_dir" || return 1
    fi

    # 6. Finalize Workspace and Hand Handoff to User
    cd "${base_dir}/${project_name}" || return 1
    echo "--- Setup Complete. Active Directory: $(pwd) ---"
  }
  ```
- Apply new env
  ```shell
  [ordinaryuser@Windows user]$ source ~/.bashrc
  ```
- Configure git env
  ```shell
  [ordinaryuser@Windows user]$ git config --global user.name "Your Name"
  [ordinaryuser@Windows user]$ git config --global user.email "your-email@example.com"
  [ordinaryuser@Windows user]$ git config --global init.defaultBranch main
  ```
