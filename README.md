# `code-server` wrapper

A simple Bash script to assist running code-server on a remote host (e.g. HPC cluster, etc).

## How to use it

- Edit the `code` script by changing the settings in the corresponding section at the beginning of the file; most importantly, change the prefix variable if you plan to install `code-server` in a location different than `~/.local`. If you plan to run `code` without a controlling terminal (e.g. via a job submission script), also replace the last line (`read -r -d '' _ </dev/tty`) with just `wait`.
- Copy `code` somewhere in your path and give it execution permissions.
- If you use a modules environment, load the modules you want to use with `code-server`.
- Just run `code`; if the script doesn't find `code-server` in the prefix (typically, on the first run), it can perform an automatic install (it needs an interactive session for that).
- Then just follow the on-screen instructions; they provide a `ssh` command to be run on your local computer and `code-server`'s access password.
- Once you run the displayed `ssh` command, you should be able to open your browser at `localhost:someport`, insert your password, and get a full VS Code environment.

Just a few notes:

- The `code-server` command used to open VS Code in the current directory, but that's no longer the case. So the first argument of the `code` command is not actually used.
- The `code` script is configured to look for a fixed password in `code-server`'s config file. If you want to use a OTP, you probably have to make some edits.
- By default, `code-server` uses HTTP; it should not be a big deal, since the connection is still securely tunneled via `ssh`. Anyway, it's possible to configure it to use HTTPS, just follow the instructions in `code-server`'s docs (https://github.com/cdr/code-server/blob/main/docs/guide.md).
- Provided that you've replaced the last line of `code` with `wait`, you can also use a submission script to run `code` in a cluster compute node. The script could just be as simple as:
  ```bash
  #!/usr/bin/env bash

  module load <yourmodules>

  while true; do code; done
  ```
  with the infinite `while` loop giving you the possibility to kill `code` and have it automatically restarted if something goes wrong, without losing the allocated compute node. I usually prefer to open a `screen` session on the login node, ask for an interactive job and run `code` from there, after which I can just detach. This has the additional benefit of being able to change the environment fed to `code-server` by just killing `code`, load different software modules, and run `code` again.

**Warnings:**

- This script is quite ugly and prone to errors.
- The port-forwarding `ssh` command works well if you're accessing a remote server directly exposed to the internet or to your local network. If this is not the case (typically for HPC clusters, etc.), you either have to manually change the proposed `ssh` command to suit your setup or you have to configure yor `~/.ssh/config` file to use cascade jump hosts, after which you can use your custom hostname as pointed out in the script. I've tested the script in a scenario `mymachine:myport -> sshentrypoint -> controlserver -> execserver:codeport` where `code-server` was running on `execserver:codeport` and it worked perfectly, so it's definitely doable. 
- I don't really plan to maintain this script at all, as I don't have any time right now. If you have troubles, you're on your own. If you want to propose improvements, please open a PR with the proposed changes.
