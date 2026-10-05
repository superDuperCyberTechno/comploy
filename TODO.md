# TODO

Potential problems found in the `comploy` executable (v2.6). Items marked FIXED were addressed in 2.6.

## 1. Critical (data loss / destructive)

### 1.1 Empty default host entry can wipe server home directory **[FIXED in 2.6]**
Config ships with `hosts['']=''` (line 28). During sync, `rsync -a --delete ./ root@${host}:${server_dir}` (line 105) -- if a host is set but `server_dir` left blank, destination becomes `root@host:` = `/root`, and `--delete` wipes everything there. Empty `host` also produces `ssh root@` (line 74) which fails, and script continues regardless.

Fix: placeholder removed; host config is validated up front (empty/relative/root `server_dir` entries are skipped or abort) and execution stops with an error when no valid hosts remain.

### 1.2 No error handling anywhere, and unconditional success message **[FIXED in 2.6]**
No `set -e`, no exit-code checks on `ssh`, `rsync`, `git push`, `composer`, or `chown`. If rsync fails (network error, missing key, permission denied), the script still runs remote `chown`/post-deploy commands, still runs `git push`, and unconditionally prints "Project synced and ready to go!". Failures are silent and misreported.

Fix: `set -euo pipefail` added; any failing ssh/rsync/composer/git command aborts before later steps run.

## 2. High (correctness)

### 2.1 Abort path leaves local project in production composer state
`composer install --no-dev --optimize-autoloader` (line 58) runs *before* the first-deploy dirty-repo abort check (line 77). On abort (`exit 1`, line 79) the dev environment is never restored -- local dev packages are gone.

### 2.2 Composer runs unconditionally
Lines 56-59 execute even when composer is not installed, `composer.lock` absent, or `hosts` empty. Failure is ignored and disaster (#1.1) still attempted.

### 2.3 `chmod 644` strips execute bits from all files server-side
Line 108. `vendor/bin/*` scripts, `artisan`, and any executable lose `+x` on every deploy. Setting `644` on files rsync just wrote with `-a` also defeats `-a`'s permission preservation.

### 2.4 Commit logic tests the wrong thing
Line 86. `[[ $(git diff --cached --exit-code) ]]` tests the captured *stdout* (diff text), discarding the exit status. It works only by accident -- empty output when nothing staged. Fragile pattern; breaks for edge cases (staged-but-identical changes).

### 2.5 `if $use_composer` / `if $use_git` execute variable contents as commands
Lines 56, 61, 70, 85, 94, 116. Works only because `true`/`false` are shell builtins; any value with spaces breaks; a typo like `use_git=ture` silently disables git (dirty tree deploys like non-git mode).

### 2.6 `git push` runs even after a failed deploy
Line 121. Push failure is ignored. Non-fast-forward or missing remote reports "all is well".

### 2.7 First-run dirty-repo check only runs when `use_git=true`
Line 70. With git disabled, uncommitted work is synced with `--delete` semantics -- no warning.

## 3. Medium

### 3.1 Hardcoded `root@` user and `~/.ssh/id_rsa`
Lines 67, 74. No ssh port, `-o` options, or StrictHostKeyChecking; first connection prompts interactively and can hang in automation. Modern systems default to `id_ed25519`, not `id_rsa`.

### 3.2 Remote command quoting is broken for paths with spaces
`server_dir` is interpolated into a single-quoted ssh argument (line 74) and unquoted inside the remote `chown`/`find` string (line 108). Any host path containing spaces or shell metacharacters breaks or injects.

### 3.3 `rsync -e "ssh -i ${key}"` breaks if the key path contains spaces
Line 105.

### 3.4 `/tmp/comploy_ignore` is a predictable, world-writable path
Lines 92-98. Symlink-attackable by another local user mid-run; never cleaned up; collides if two comploy projects run concurrently.

### 3.5 Ignore patterns match at any depth
Rsync exclude entries without a leading `/` match basenames everywhere -- a modified `config.php` (line 96) excludes *every* `config.php` in the tree from the deploy.

### 3.6 `declare -A` requires bash 4+
Line 2; fails on bash 3.2 (default macOS). README declares Linux-only, so a portability note only.

### 3.7 `ssh` failure during first-run detection is treated as "not first run" **[FIXED in 2.6]**
Line 74. The dirty-repo abort is silently skipped, and sync proceeds into a dead-end.

### 3.8 Multiple hosts are non-atomic
Line 102: a failure in host 2 leaves host 1 already deployed with no error report.

### 3.9 `chown -R www-data` + full `find` chmod on every deploy
Line 108. Resets server-side permission tweaks, churns the whole tree, and is slow on large projects.

### 3.10 Symlink hazards
`-a` preserves local symlinks; with `--delete`, a missing/mismatched local symlink (e.g., Laravel `storage`) can orphan or delete the server-side link target.

### 3.11 `cd "$SCRIPT_PATH"` with no error check **[FIXED in 2.6]**
Line 53. Script must live exactly in repo root; deployed content silently wrong otherwise.

### 3.12 Post-deployment commands exit status ignored **[FIXED in 2.6]**
Line 112. A failing remote command is never reported.

### 3.13 No dry-run or confirmation before destructive `--delete` sync **[FIXED in 2.6]**
One wrong config key wipes a remote directory (#1.1).

Fix: sync is refused unless every `server_dir` is a non-empty absolute path other than `/`.

### 3.14 `ignored_files` is space-separated
Line 22. Cannot ignore any filename containing a space.