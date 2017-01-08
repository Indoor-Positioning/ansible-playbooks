# Ansible playbooks for expiremental setup of [Eddystone-UID](https://github.com/google/eddystone/tree/master/eddystone-uid) beacons via Raspberry Pi

This repo includes 3 ansible playbooks.

* [setup-ansible-user.yml](setup-ansible-user.yml)
* [site.yml](site.yml)
* [stop-advertising.yml](stop-advertising.yml)

##### Playbook variables (applicable to site.yml only)

Eddystone-UID **namespace**:
```yaml 
namespace: "01 23 45 67 89 ab cd ef 01 23"
```

The 10-byte namespace - most probably it ll be common between the rpis of the same group.

Eddystone-UID **instance**:
```yaml 
instance: "00 00 00 00 00 00"
```

The 6-byte pi-specific instance 

Tx Power Level **calibrated_power**
```yaml
calibrated_power: "00"
```

The signed 8 bit integer that corresponds to the [Transmit Power Level characteristic](https://www.bluetooth.com/specifications/gatt/viewer?attributeXmlFile=org.bluetooth.characteristic.tx_power_level.xml&u=org.bluetooth.characteristic.tx_power_level.xml)
and represents the current transmit power level in dBm. the level ranges from -100 dBm to +20 dBm.


##### [Setup ansible user](setup-ansible-user.yml)

This playbook creates the ansible user on the Raspberry pi, and gives sudo permissions without asking
for password. Pi user is assumed for this play, as this is the default user on Raspbian.

```bash
ansible-playbook -i hosts setup-ansible-user.yml 
```

(Modify hosts accordingly, based on the ip addresses of the raspberry pis).

##### [Main playbook (site.yml)](site.yml)

This playbook sets up the raspberry pis to le advertising mode, and sends le advertisements based on the
namespace and instance that are defined via host variables.

We first check whether pi is ble-capable (e.g bluez version >= 4.99), then bring the hci0 interface
up, and finally we start broadcasting non-connectable eddystone-uid ads.

The Ranging Data (Calibrated Tx power at 0 m) is defaulted to `00`, and is meant to be overriden based
on the RSSI measured at 0 m.
The namespace is meant to be common between the pis, and is defined under [group_vars/pi.yml]([group_vars/pi.yml])
The instance is host specific, and is defined under [host_vars/<pi_ip_address>](host_vars/192.168.1.9.yml)

Execution:

```bash
ansible-playbook -i hosts site.yml 
```

##### [Stop advertising](stop-advertising.yml)

This playbook stops the le advertising.

```bash
ansible-playbook -i hosts stop-advertising.yml 
```