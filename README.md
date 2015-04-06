## Nagios Check for Sub10 Systems Liberator E1000

This Nagios/Icinga Check provides the ability to query Sub10 Systems Liberator E1000 devices for status and parameters. It will output performance data to monitor reception quality and errors for tools like pnp4nagios. A event handler script for Rebooting a device is also provided. 
Implementation is in Python. You will need Python libraries pnp4nagios and pysnmp as dependencies.

You need to enable the SNMP Agent on the Sub10 device and set a SNMP Read community. For using the reboot script, you need to also set the SNMP write community.

### Installation (manual using pip) on your Nagios Host
```
pip install nagiosplugin pysnmp
python setup.py install
ln -s /usr/bin/check_sub10_e1000 /usr/lib/nagios/plugins/check_sub10_e1000
ln -s reboot_sub10_e1000 /usr/share/nagios3/plugins/eventhandlers/reboot_sub10_e1000
```

### Usage example

Nagios Plugin called manually:

```
./check_sub10_e1000 -H 1.2.3.4 -C public 
```

See check_sub10_e1000 -h for additional command line arguments. Use -vvv to get Debug Output including serial, terminal name, link name, firmware data and additional information.

Reboot event handler script called manually:
```
./reboot_sub10_e1000 -H 1.2.3.4 -C private DOWN HARD 1 
```

### Nagios Integration

Define the commands for Nagios checks and include it in the service definitions:

```
define command {
        command_name    check_sub10_e1000
        command_line    /usr/lib/nagios/plugins/check_sub10_e1000 -C $ARG1$ -H $HOSTADDRESS$
}
define service {
	use			generic-service-perfdata
	hostgroup_name		sub10g
	service_description	check_sub10_e1000
	check_command		check_sub10_e1000!SNMP_COMMUNITY
}
```

To use the event handler script to reboot a Sub10 device, define the reboot command and use it on the child device:

```
define command {
	command_name	reboot_sub10_e1000_sub10-device
	command_line	/usr/share/nagios3/plugins/eventhandlers/reboot_sub10_e1000 -H FIXME -C private $HOSTSTATE$ $HOSTSTATETYPE$ $HOSTATTEMPT$
}
define host{
        us				generic-host
        host_name			some-child-host
        parents				sub10-device
        address				FIXME
        event_handler_enabled		1
        event_handler			reboot_sub10_e1000_sub10-device
}

```
