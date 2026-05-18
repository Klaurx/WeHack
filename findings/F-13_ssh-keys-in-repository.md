# F-13: SSH Public Keys Committed to Source Repository

## What was observed

The `server-template/checklist` file committed in the 试玩测试 workspace contains two SSH public keys embedded in the provisioning notes:

The first key is attributed to `lowkey@lowkey.local` — the developer's local machine. The second key is attributed to `root@gamebackstage` — the Jenkins host itself.

Both keys are RSA keys. Both are committed to the repository alongside the instruction to add them to the `~/.ssh/authorized_keys` file on newly provisioned production servers.

## Why this matters

SSH public keys are not secret by themselves — they are designed to be distributed. The specific concern here is what these keys reveal and enable in the current context.

The Jenkins host key (`root@gamebackstage`) being in the authorized_keys of production servers is what enables the root SSH deployment described in F-01. Its presence in the repository confirms this access was provisioned intentionally and systematically across all servers built from this template.

If the corresponding private keys have been exposed anywhere — in another file in these repositories, in a Jenkins credential store that has been accessed, or on a compromised host — then every server built from this template has been silently accessible.

The developer key (`lowkey@lowkey.local`) indicates the developer also has direct root SSH access to production servers from their personal machine.

## What needs to change

- Audit all servers provisioned from this template to determine which ones have these keys in their authorized_keys.
- Remove both keys from authorized_keys on all production servers.
- If SSH access to production is still required, provision new keys that are not committed to any repository, restricted to specific source IPs, and not for the root user.
- Remove the checklist file from the repository and rewrite git history.
