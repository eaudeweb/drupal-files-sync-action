# drupal-files-sync-action

GitHub composite action that syncs public files from a PROD server to a TEST server using SSH agent forwarding and ephemeral keys.

## How it works

1. Writes permanent SSH keys (runner → PROD, runner → TEST) from inputs
2. Generates an ephemeral ed25519 key pair unique to each run
3. Adds the ephemeral public key to TEST's `authorized_keys`
4. Checks available disk space on TEST — requires at least 5% of total disk to remain free after transfer
5. Runs `rsync` from PROD → TEST via SSH agent forwarding (the private key never touches PROD)
6. Cleans up: removes the ephemeral key from TEST and all keys from the runner

## Usage

```yaml
- uses: eaudeweb/drupal-files-sync-action@1.x
  with:
    source_files_dir: ${{ vars.PROD_PUBLIC_FILES_DIR }}
    target_files_dir: ${{ vars.TEST_PUBLIC_FILES_DIR }}
    prod_ssh_key:     ${{ secrets.PROD_SSH_KEY }}
    prod_ssh_host:    ${{ secrets.PROD_SSH_HOST }}
    prod_ssh_user:    ${{ secrets.PROD_SSH_USER }}
    test_ssh_key:     ${{ secrets.TEST_SSH_KEY }}
    test_ssh_host:    ${{ secrets.TEST_SSH_HOST }}
    test_ssh_user:    ${{ secrets.TEST_SSH_USER }}
```

## Inputs

| Input | Required | Description |
|---|---|---|
| `source_files_dir` | yes | Absolute path to the files directory on PROD |
| `target_files_dir` | yes | Absolute path to the files directory on TEST |
| `prod_ssh_key` | yes | SSH private key for PROD server |
| `prod_ssh_host` | yes | PROD server hostname or IP |
| `prod_ssh_user` | yes | SSH user for PROD server |
| `test_ssh_key` | yes | SSH private key for TEST server |
| `test_ssh_host` | yes | TEST server hostname or IP |
| `test_ssh_user` | yes | SSH user for TEST server |

## Server requirements

- `web` user must be in the `nginx` group on both servers
- Files directory must have `2775` permissions (`drwxrwsr-x`)
- The runner's IP must be allowed to SSH to both servers
- PROD's IP must be allowed to SSH to TEST on port 22
