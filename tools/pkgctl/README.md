Usage
=====

Example [gemini-cli configuration](https://geminicli.com/docs/reference/configuration/)

```json
{
  "tools": {
    "discoveryCommand": "/opt/cemi/bin/pkgctl --schema",
    "callCommand": "/opt/cemi/bin/sudo_pkgctl"
  }
}
```

Example `sudo` rule

```bash
echo "$USER ALL=(root) NOPASSWD: /opt/cemi/bin/pkgctl" > /etc/sudoers.d/"$USER"
```
