# drupal-files-sync-action

GitHub composite action that syncs public files from a PROD server to a TEST server using SSH agent forwarding and ephemeral keys.

## How it works

1. Write permanent SSH keys (runner → PROD, runner → TEST) from inputs
2. Generate an ephemeral ed25519 key unique to each run
3. Add the ephemeral public key to TEST's `authorized_keys`
4. Validate available disk space on TEST — requires at least 5% of total disk to remain free after transfer
5. `rsync` from PROD → TEST via SSH agent forwarding (the private key never touches PROD)
6. Remove the ephemeral key from TEST and all keys from the runner

## Usage

Note: Example is using GitHub **variables**.

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

| Input              | Required | Description                                                        |
|--------------------|----------|--------------------------------------------------------------------|
| `source_files_dir` | yes      | Absolute path to the files directory on PROD (without ending `/`)  |
| `target_files_dir` | yes      | Absolute path to the files directory on TEST (without ending `/`)  |
| `prod_ssh_key`     | yes      | Permanent SSH private key for PROD server (must be configured)     |
| `prod_ssh_host`    | yes      | PROD server hostname or IP                                         |
| `prod_ssh_user`    | yes      | SSH user for PROD server                                           |
| `test_ssh_key`     | yes      | Permanent SSH private key for TEST server (must be configured)     |
| `test_ssh_host`    | yes      | TEST server hostname or IP                                         |
| `test_ssh_user`    | yes      | SSH user for TEST server                                           |

## Server requirements

- For each PROD / TEST configure permanent SSH keys (and add public key in `authorized_keys`)
- The runner's IP must be allowed to SSH to both servers
- PROD's IP must be allowed to SSH to TEST on port 22
- `*_SSH_USER` user must be in the `nginx` / `apache` group on both servers to access files
- TEST files directory must have correct ownership and permissions (`2775 / drwxrwsr-x`) for rsync to write properly.
```bash
chown -R web:nginx files/
chmod -R g+w files/
chmod g+s files/
```
