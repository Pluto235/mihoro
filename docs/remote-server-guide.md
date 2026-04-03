# Remote Server Guide

This fork is used as a practical Mihomo manager for long-running Linux servers accessed over SSH.
It is still based on upstream `mihoro`, but the workflow here is intentionally optimized for a safer remote setup.

## Deployment Model

A typical deployment on a remote Linux server looks like this:

- `mihoro` binary: `~/.local/bin/mihoro`
- `mihomo` core: `~/.local/bin/mihomo`
- `mihoro` config: `~/.config/mihoro.toml`
- generated `mihomo` config root: `~/.config/mihomo/`
- user service: `~/.config/systemd/user/mihomo.service`

The recommended model for remote hosts is:

- bind local proxy ports to `127.0.0.1`
- bind `external_controller` to `127.0.0.1:9090`
- access the controller or dashboard through SSH port forwarding instead of exposing it to the public internet

## Suggested Config Defaults

Example:

```toml
remote_config_url = "https://example.com/subscription?flag=clash"
mihomo_channel = "stable"

[mihomo_config]
port = 7891
socks_port = 7892
mixed_port = 7890
allow_lan = false
bind_address = "127.0.0.1"
external_controller = "127.0.0.1:9090"
external_ui = "ui"
secret = "replace-with-your-own-secret"
```

Do not commit a real subscription URL, secret, token, or controller password to a public repository.

## Service Operations

Start or restart after setup:

```bash
mihoro setup
mihoro restart
```

Common commands:

```bash
mihoro status
mihoro start
mihoro stop
mihoro restart
mihoro log
mihoro apply
mihoro update --config
mihoro update --core
mihoro update --all
```

## Proxy For The Current Shell

Enable proxy for the current shell session:

```bash
eval "$(mihoro proxy export)"
```

Disable proxy for the current shell session:

```bash
eval "$(mihoro proxy unset)"
```

If you prefer shorter wrappers in your shell profile:

```bash
proxy_on() {
    eval "$(mihoro proxy export)"
}

proxy_off() {
    eval "$(mihoro proxy unset)"
}
```

## Auto Enable Proxy After SSH Login

For remote server workflows, it is convenient to enable the proxy only for interactive shells.
That keeps `scp`, `rsync`, cron jobs, and non-interactive scripts unaffected.

Example for `~/.bashrc`:

```bash
# Ensure user-local binaries are available in interactive bash shells.
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi

case $- in
    *i*) ;;
      *) return;;
esac

if command -v mihoro >/dev/null 2>&1; then
    eval "$(mihoro proxy export)" >/dev/null 2>&1 || true
fi
```

## Dashboard Over SSH Port Forwarding

A remote server should not expose Mihomo's controller publicly by default.
Use a dashboard together with SSH local forwarding instead.

If `external_controller = "127.0.0.1:9090"` and `external_ui = "ui"`, you can forward the controller locally:

```bash
ssh -L 9090:127.0.0.1:9090 user@server
```

Then open this URL in your local browser:

```text
http://127.0.0.1:9090/ui/
```

If the dashboard asks for backend information:

- Backend URL: `http://127.0.0.1:9090`
- Secret: the value configured in `mihomo` / `mihoro.toml`

## Notes For This Fork

This fork documents a real server workflow:

- safer local-only controller binding
- shell-level proxy helpers for SSH sessions
- dashboard access over SSH port forwarding
- operator-facing documentation for day-to-day remote administration

These are operational customizations on top of upstream `mihoro`, not a rewrite of the original project.
