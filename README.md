# Zabbix Agent 2 – LVM Plugin

A Linux loadable plugin for Zabbix Agent 2 that collects LVM Physical Volumes (PV), Logical Volumes (LV), and Volume Groups (VG).

## Tested environments

The plugin has been tested on the following platform:

- **Operating System:** Red Hat Enterprise Linux 10
- **Monitoring System:** Zabbix Agent 2 – 8.0 LTS
- **Architecture:** Linux x86_64 (AMD64)

## Requirements

- Zabbix Agent 2 (8.0 LTS).
- LVM2 utilities: `pvs`, `lvs`, and `vgs`.
- If Agent 2 runs as an unprivileged user, configure non-interactive sudo access for the three read-only LVM reporting commands.

## Install the plugin

Copy the downloaded 'zabbix-agent2-plugin-lvm' file to the target server.

```bash
install -D -o root -g root -m 755 \
  zabbix-agent2-plugin-lvm \
  /usr/lib/zabbix/agent2/zabbix-agent2-plugin-lvm

mkdir -p /etc/zabbix/zabbix_agent2.d/plugins.d
cat > /etc/zabbix/zabbix_agent2.d/plugins.d/lvm.conf <<'CONF'
Plugins.LVM.System.Path=/usr/lib/zabbix/agent2/zabbix-agent2-plugin-lvm
CONF
```

Ensure the main `/etc/zabbix/zabbix_agent2.conf` includes:

```ini
Include=/etc/zabbix/zabbix_agent2.d/plugins.d/*.conf
```

## Permissions

If Agent 2 runs as the `zabbix` user, the plugin invokes each LVM report command through `sudo -n`. Create a restricted sudoers entry:

```bash
visudo -f /etc/sudoers.d/zabbix-lvm
```

Add:

```sudoers
zabbix ALL=(root) NOPASSWD: /usr/sbin/pvs, /usr/sbin/lvs, /usr/sbin/vgs
```

Confirm the paths with `command -v pvs lvs vgs`, validate with `visudo -cf /etc/sudoers.d/zabbix-lvm`, and test each command as the `zabbix` user. The rule permits the three report utilities; the plugin does not invoke LVM modification commands.

## Restart and test

```bash
systemctl restart zabbix-agent2
systemctl status zabbix-agent2
journalctl -u zabbix-agent2 -f
```

Test the single master item:

```bash
zabbix_agent2 -t 'lvm.get'
```

The result should be a JSON object containing `pvs`, `lvs`, and `vgs` arrays. All three dependent discovery rules and all dependent item prototypes are populated from this one master item using JSONPath preprocessing.
