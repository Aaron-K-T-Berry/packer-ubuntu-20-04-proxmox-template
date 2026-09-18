# Packer Ubuntu 20.04 Proxmox template

This repo has a Packer HCL template that creates a cloud-init enabled Ubuntu Server 20.04 Proxmox template using live-server autoinstall.

This repo has [Taskfile](https://taskfile.dev/#/) support to make running some of the common commands easier.

**See also:** [Ubuntu 18.04](https://github.com/Aaron-K-T-Berry/packer-ubuntu-proxmox-template) (legacy preseed) · [Ubuntu 24.04](https://github.com/Aaron-K-T-Berry/packer-ubuntu-24-04-proxmox-template) — see [Sibling templates](#sibling-templates) for install-path differences.

Ubuntu 20.04 standard support ended in April 2025. This is a working example of a Packer Proxmox build.

## How installation works

- Uses the **Ubuntu live-server ISO** with **Subiquity autoinstall** (NoCloud). Packer templates [`http/user-data.pkrtpl`](./http/user-data.pkrtpl) and serves it as `/user-data` (exact path), plus [`http/meta-data`](./http/meta-data).
- Autoinstall needs a **SHA-512** password hash in `ssh_password_hash`, plus matching plaintext in `ssh_password` for Packer SSH. See [Autoinstall password hash](#autoinstall-password-hash).
- The builder sets **`cloud_init = true`**. You do **not** need a `qm set` post-processor. [`files/99-pve.cfg`](./files/99-pve.cfg) prefers ConfigDrive/NoCloud for Proxmox.
- This is **not** the 18.04 debian-installer preseed path. Example defaults: about **2G** RAM, **20G** disk, `local-lvm` for disk and cloud-init storage.

## Sibling templates

Other Ubuntu Proxmox Packer templates in this family:

| Version | Install path | Repository |
| --- | --- | --- |
| [18.04](https://github.com/Aaron-K-T-Berry/packer-ubuntu-proxmox-template) | Classic server ISO + debian-installer preseed; cloud-init via `qm set` | [packer-ubuntu-proxmox-template](https://github.com/Aaron-K-T-Berry/packer-ubuntu-proxmox-template) |
| **20.04 (this repo)** | live-server + Subiquity autoinstall; builder-native cloud-init | [packer-ubuntu-20-04-proxmox-template](https://github.com/Aaron-K-T-Berry/packer-ubuntu-20-04-proxmox-template) |
| [24.04](https://github.com/Aaron-K-T-Berry/packer-ubuntu-24-04-proxmox-template) | Same autoinstall pattern as this repo, with higher RAM (4G) and a firstboot helper | [packer-ubuntu-24-04-proxmox-template](https://github.com/Aaron-K-T-Berry/packer-ubuntu-24-04-proxmox-template) |

## Creating an Ubuntu Proxmox template

You can run the template with the following methods:

- [The simple way](#the-simple-way)
- [The manual way](#the-manual-way)

This template has required configuration variables in `example-vars.pkrvars.hcl`. Copy that file, fill in the values for your environment, and pass it to Packer with `-var-file`.

You need the [`Ubuntu 20.04 live server ISO`](https://releases.ubuntu.com/20.04/) already available on the PVE host. If you have [`task`](https://taskfile.dev/#/) installed you can download it locally with `task ubuntu-20-iso` and then upload it to the PVE host.

The ISO on the node must match the `iso` variable, for example `local:iso/ubuntu-20.04.6-live-server-amd64.iso`.

### The simple way

This repo uses a [Taskfile](https://taskfile.dev/#/) for common commands. You can find installation instructions for `task` [here](https://taskfile.dev/#/installation).

1. Configure template variables for your setup

   ```shell
   $ task init
   cp ./example-vars.pkrvars.hcl ./config.pkrvars.hcl
   ```

   After the task has completed, fill out `config.pkrvars.hcl` for your environment. See the [variables](#variables) section for more details.

2. Check your template file is valid

   ```shell
   $ task validate
   packer init .
   packer validate -var-file="./config.pkrvars.hcl" .
   The configuration is valid.
   ```

3. Build your Proxmox template

   ```shell
   $ task build
   packer init .
   packer build -var-file="./config.pkrvars.hcl" .
   ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: output will be in this color.

   ==> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: Creating VM
   ==> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: Starting VM
   ==> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: Starting HTTP server on port 8000

   ...

   Build 'ubuntu-server-focal.proxmox-iso.ubuntu-server-focal' finished.

   ==> Builds finished. The artifacts of successful builds are:
   --> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: A template was created: 4000
   ```

4. Your template should now be available on your PVE host

### The manual way

1. Configure template variables for your setup

   ```shell
   cp ./example-vars.pkrvars.hcl ./config.pkrvars.hcl
   ```

   After copying, fill out `config.pkrvars.hcl` for your environment. See the [variables](#variables) section for more details.

2. Check your template file is valid

   ```shell
   $ packer init .
   $ packer validate -var-file="./config.pkrvars.hcl" .
   The configuration is valid.
   ```

3. Build your Proxmox template

   ```shell
   $ packer build -var-file="./config.pkrvars.hcl" .
   ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: output will be in this color.

   ==> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: Creating VM
   ==> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: Starting VM
   ==> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: Starting HTTP server on port 8000

   ...

   Build 'ubuntu-server-focal.proxmox-iso.ubuntu-server-focal' finished.

   ==> Builds finished. The artifacts of successful builds are:
   --> ubuntu-server-focal.proxmox-iso.ubuntu-server-focal: A template was created: 4000
   ```

4. Your template should now be ready on your PVE host

If you had no errors building your image you should now have a Proxmox template that is cloud-init enabled. The template name is the `template_name` value plus a timestamp.

## Variables

| Variable | Description | Example |
| --- | --- | --- |
| `proxmox_host` | Hostname and port of the PVE API, without a scheme | `100.0.0.100:8006`, `pve.example.com:8006` |
| `proxmox_node_name` | Node the template will be created on | `pve-01` |
| `proxmox_api_user` | API user plus login realm | `root@pam` |
| `proxmox_api_password` | Password for that API user | `password` |
| `proxmox_insecure_skip_tls_verify` | Skip TLS verification for the Proxmox API | `true` |
| `template_name` | Base name for the VM and resulting template. Packer appends a timestamp | `ubuntu-20-04` |
| `template_description` | Description applied to the Proxmox template | `Ubuntu 20.04, generated by Packer` |
| `ssh_username` | Default guest user created by autoinstall. Packer SSH uses the same user | `packer` |
| `ssh_password` | Plaintext password Packer uses for SSH. Must match `ssh_password_hash` | `packer` |
| `ssh_password_hash` | SHA-512 hash of `ssh_password` for autoinstall identity | see `example-vars.pkrvars.hcl` |
| `hostname` | Hostname set during autoinstall | `ubuntu-20-04-cloudinit` |
| `vmid` | ID of the VM used to build the template | `4000` |
| `locale` | Installer locale | `en_US` |
| `timezone` | Guest timezone | `UTC` |
| `cores` | vCPU cores | `2` |
| `sockets` | CPU sockets | `1` |
| `memory` | RAM in MiB. Ubuntu 20.04 live-server autoinstall needs about 2G | `2048` |
| `disk_size` | OS disk size | `20G` |
| `datastore` | Storage pool for the OS disk | `local-lvm` |
| `cloud_init_storage_pool` | Storage pool for the cloud-init drive | `local-lvm` |
| `network_bridge` | Linux bridge attached to the template NIC | `vmbr0` |
| `iso` | PVE path to the Ubuntu 20.04 live-server ISO | `local:iso/ubuntu-20.04.6-live-server-amd64.iso` |
| `ssh_timeout` | How long Packer waits for SSH after autoinstall | `90m` |
| `ssh_handshake_attempts` | SSH handshake retry attempts | `20` |
| `http_bind_address` | Address Packer's autoinstall HTTP server binds to | `0.0.0.0` |

### Autoinstall password hash

Subiquity autoinstall needs a hashed password, not the plaintext value. The example hash is for the password `packer`. Generate a replacement with:

```shell
mkpasswd -m sha-512 packer
# or
openssl passwd -6 packer
```

Put the hash in `ssh_password_hash` and keep `ssh_password` as the matching plaintext so Packer can SSH after install.

You can also change the guest user by updating `ssh_username` in your var file. The autoinstall `user-data` template reads those variables.

## Troubleshooting

### The installer stays on the language screen, or Packer times out waiting for SSH

The guest must be able to download autoinstall files from Packer's HTTP server. The boot command uses Packer's advertised `{{ .HTTPIP }}:{{ .HTTPPort }}`. If that address is not reachable from the PVE node (container IP, VPN interface, firewall), autoinstall never starts.

- Confirm `http_bind_address` is `0.0.0.0` so Packer listens on all interfaces
- Open the HTTP port Packer prints at start (`Starting HTTP server on port ...`) from the PVE node to the machine running Packer
- If Packer picked an IP the VM cannot route to, run Packer on a host the PVE bridge can reach

### The ISO is missing on the node

Upload `ubuntu-20.04.6-live-server-amd64.iso` to the storage referenced by `iso` before building. This template does not download the ISO onto PVE for you.

### SSH never comes up after install

Confirm `ssh_password` matches `ssh_password_hash`, that `memory` is at least `2048`, and that the live-server ISO is 20.04 (Subiquity autoinstall), not the legacy debian-installer ISO.

## Notes

- This template is intended for `ubuntu-20.04.6-live-server-amd64.iso` (live-server only; classic debian-installer ISOs will not work)
- You can change the OS install by editing [`http/user-data.pkrtpl`](./http/user-data.pkrtpl)
- The builder enables cloud-init on the Proxmox template directly. You do not need a separate `qm set` post-processor
- Example defaults use about **2G** RAM; keep `memory` at least `2048` for live-server autoinstall
- `config.pkrvars.hcl` is gitignored. Keep real API passwords out of git
