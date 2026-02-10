---
title: "How to Netmiko - 5 minutes to all devices"
date: 2021-05-06
categories: 
  - "uncategorized-en"
tags: 
  - "netmiko"
  - "python"
---

Start using Netmiko is easier than you might think.

Going from being a pure network guy to network automation with software development skills is NOT easy, and it is a long shot.

Usually, people (like me) start with Ansible just because it is easier when you don't know much about Python. Plus you can get some things done pretty fast.

But at some point when you already acquired some Python skills you need to come back to review fundamentals to close the loop. In this case, I will start with [Netmiko](https://github.com/ktbyers/netmiko). Thanks Kirk for this awesome library! He also offers a [free Python for network engineers training course](https://pynet.twb-tech.com/email-signup.html) <- This is a must!

I will make this post hipper short and concrete with some examples to show you that it is indeed very easy. The code is fully commented on to facilitate understanding.

_**We are going in an incremental way adding to the scripts from 0 to throw commands to all your devices!**_

You can find the code in this [repository](https://github.com/aegiacometti/netmiko_5m_to_all).

Let's go! 

**1.- basic.py:** code to connect to a single device

```python
# Basic code to run a commmand in a device
# Just modify your "device_info" credentials variable
import netmiko
import sys
# device dictionary expected by netmiko library
device_info = {
    "device_type": "cisco_ios",
    "username": "cisco",
    "password": "cisco",
    "secret": "cisco",
}
# get host IP and command to execute from CLI
# host = sys.argv[1]
# command = ' '.join(sys.argv[2:])
host = input('Enter host IP: ')
command = input('Enter command to run: ')
# add the IP address of the host to the dictonary
device_info['host'] = host
# Start SSH session and login to device
ssh_session = netmiko.ConnectHandler(**device_info)
# Enter enable mode
ssh_session.enable()
# Send command to device and store it in a variable
result = ssh_session.send_command(command)
# End SSH session
ssh_session.disconnect()
# Show results of the command
print(result)
```

_Output example:_

```basic
adrian@adrian:~$ python3 1_basic.py
Enter host IP: 10.100.200.1
Enter command to run: show clock
.14:05:46.542 UTC Thu May 6 2021
```

**2.- intermediate.py:** code to connect to multiple devices

```python
# Just modify the file inventory.yml with your info
# Change import settings
import yaml
import sys
import time
from netmiko import ConnectHandler, NetmikoTimeoutException, NetmikoAuthenticationException
# Reusable send command function
def send_command(dev: dict, cmd: str) -> str:
    """
    Send command to device using Netmiko
    :param dev: device info
    :param cmd: command to execute
    :return: Command output from device
    """
    hostname = device['hostname']
    # remove key hostname from dictionary since it is not expected/valid for netmiko
    del dev['hostname']
    try:
        # Use context manager to open and close the SSH session
        with ConnectHandler(**dev) as ssh:
            ssh.enable()
            output = ssh.send_command(cmd)
        output = output.strip()
    except NetmikoTimeoutException:
        output = 'Connection timed out'
    except NetmikoAuthenticationException:
        output = 'Authentication failed'
    # re add key for future use
    dev['hostname'] = hostname
    return output
if __name__ == "__main__":
    # get parameters from CLI
    # command = ' '.join(sys.argv[2:])
    command = input('nCommand to run: ')
    # Load devices from file
    with open('inventory.yml') as f:
        inventory = yaml.safe_load(f.read())
    # get the common variables for all devices
    credentials = inventory['common_vars']
    devices_counter = len(inventory['hosts'])
    print(f'nExecuting command: {command}n')
    # Start timer variable
    execution_start_timer = time.perf_counter()
    # Loop to repeat the command in all the inventory
    for device in inventory['hosts']:
        # update the device dictionary with the credentials and send command
        device.update(credentials)
        print('*** host: {} - ip: {}'.format(device['hostname'], device['host']))
        # send command and store results
        result = send_command(device, command)
        # show result
        print(f'{result}n')
    # Get and print finishing time
    elapsed_time = time.perf_counter() - execution_start_timer
    print(f"n"{command}" executed in {devices_counter} devices in {elapsed_time:0.2f} seconds.n")
```

**_inventory.yml_** example.

If you need different types of devices just move all the "_**device\_type**_" key right next to the devices. This is the [supported device matrix](https://github.com/ktbyers/netmiko/blob/develop/PLATFORMS.md).

```javascript
common_vars:
  username: cisco
  password: cisco
  device_type: cisco_ios
hosts:
  - hostname: site1-access
    host: 10.100.200.1
  - hostname: site1-core
    host: 10.100.12.1
  - hostname: isp-pe
    host: 10.0.12.1
  - hostname: site2-access
    host: 10.101.23.2
  - hostname: site2-core
    host: 10.0.13.1
  - hostname: isp-internet
    host: 10.0.14.1
```

_Output example:_

```basic
adrian@adrian:~$ python3 2_intermediate.py
Command to run: show clock
Executing command: show clock
*** host: site1-access - ip: 10.100.200.1
.14:11:08.962 UTC Thu May 6 2021
*** host: site1-core - ip: 10.100.12.1
.14:11:22.887 UTC Thu May 6 2021
*** host: isp-pe - ip: 10.0.12.1
.14:11:17.092 UTC Thu May 6 2021
*** host: site2-access - ip: 10.101.23.2
.14:11:22.145 UTC Thu May 6 2021
*** host: site2-core - ip: 10.0.13.1
.14:11:26.544 UTC Thu May 6 2021
*** host: isp-internet - ip: 10.0.14.1
.14:11:42.147 UTC Thu May 6 2021
"show clock" executed in 6 devices in 27.38 seconds.
```

**3.- complete.py:** we add a filter string to filter devices by IP or hostname, full or partial. 

```python
# Just modify your inventory.yml file
# Change import settings
import yaml
import sys
import time
from netmiko import ConnectHandler, NetmikoTimeoutException, NetmikoAuthenticationException
# Reusable send command function
def send_command(dev: dict, cmd: str) -> str:
    """
    Send command to device using Netmiko
    :param dev: device info
    :param cmd: command to execute
    :return: Command output from device
    """
    hostname = dev['hostname']
    # remove key hostname from dictionary since it is not expected/valid for netmiko
    del dev['hostname']
    try:
        # Use context manager to open and close the SSH session
        with ConnectHandler(**dev) as ssh:
            ssh.enable()
            output = ssh.send_command(cmd)
        output = output.strip()
    except NetmikoTimeoutException:
        output = 'Connection timed out'
    except NetmikoAuthenticationException:
        output = 'Authentication failed'
    # re add key for future use
    dev['hostname'] = hostname
    return output
def get_devices(device_filter: str) -> dict:
    """
    Get match devices from inventory bases on name or IP
    :param device_filter: string to look for
    :return: matching inventory
    """
    with open('inventory.yml') as f:
        inventory = yaml.safe_load(f.read())
    matched_devices = []
    if device_filter != 'all':
        for device in inventory['hosts']:
            if device_filter in device['hostname'] or device_filter in device['host']:
                matched_devices.append(device)
        inventory['hosts'] = matched_devices
    return inventory
if __name__ == "__main__":
    # get parameters from CLI. Choose the following 2 lines OR the next 2
    # device_filter = sys.argv[1]
    # command = ' '.join(sys.argv[2:])
    device_filter = input('nSpecify device filter: ')
    command = input('nCommand to run: ')
    # Load devices from file with the filter
    inventory = get_devices(device_filter)
    # get the common variables for all devices
    credentials = inventory['common_vars']
    devices_counter = len(inventory['hosts'])
    print(f'nExecuting command: {command}n')
    # Start timer variable
    execution_start_timer = time.perf_counter()
    # Loop to repeat the command in all the inventory
    for device in inventory['hosts']:
        # update the device dictionary with the credentials and send command
        device.update(credentials)
        print('*** host: {} - ip: {}'.format(device['hostname'], device['host']))
        # send command and store results
        result = send_command(device, command)
        # show result
        print(f'{result}n')
    # Get and print finishing time
    elapsed_time = time.perf_counter() - execution_start_timer
    print(f"n"{command}" executed in {devices_counter} devices in {elapsed_time:0.2f} seconds.n")
```

_Output example:_

```basic
adrian@adrian:~$ python3 3_complete.py 
Specify device filter: isp
Command to run: show clock
Executing command: show clock
*** host: isp-pe - ip: 10.0.12.1
.14:24:12.325 UTC Thu May 6 2021
*** host: isp-internet - ip: 10.0.14.1
.14:24:26.883 UTC Thu May 6 2021
"show clock" executed in 2 devices in 11.10 seconds.
```

**4.- bonus.yml:** add a print of matched devices, a confirmation, and stay in a loop to issue commands to the same list of devices

```python
# Just modify your inventory.yml file
# Change import settings
import yaml
import sys
import time
from netmiko import ConnectHandler, NetmikoTimeoutException, NetmikoAuthenticationException
# Reusable send command function
def send_command(dev: dict, cmd: str) -> str:
    """
    Send command to device using Netmiko
    :param dev: device info
    :param cmd: command to execute
    :return: Command output from device
    """
    hostname = dev['hostname']
    # remove key hostname from dictionary since it is not expected/valid for netmiko
    del dev['hostname']
    try:
        # Use context manager to open and close the SSH session
        with ConnectHandler(**dev) as ssh:
            ssh.enable()
            output = ssh.send_command(cmd)
        dev['hostname'] = hostname
        output = output.strip()
    except NetmikoTimeoutException:
        output = 'Connection timed out'
    except NetmikoAuthenticationException:
        output =  'Authentication failed'
    # re add key for future use
    dev['hostname'] = hostname
    return output
def get_devices(device_filter: str) -> dict:
    """
    Get match devices from inventory bases on name or IP
    :param device_filter: string to look for
    :return: matching inventory
    """
    with open('inventory.yml') as f:
        inventory = yaml.safe_load(f.read())
    matched_devices = []
    if device_filter != 'all':
        for device in inventory['hosts']:
            if device_filter in device['hostname'] or device_filter in device['host']:
                matched_devices.append(device)
        inventory['hosts'] = matched_devices
    # Show matched inventory and confirm
    text = 'nMatched inventory'
    print('{}n{}'.format(text, '*' * len(text)))
    for device in inventory['hosts']:
        print('* host: {} - ip: {}'.format(device['hostname'], device['host']))
    confirm = input('nPlease confirm (y/n): ')
    if confirm.lower() != 'y':
        sys.exit()
    return inventory
if __name__ == "__main__":
    # Type device filter by IP or hostname. Partial values or full. Optionally 'all'
    device_filter = input('nSpecify device filter: ')
    # Load devices from file with the filter and display matching device
    inventory = get_devices(device_filter)
    # get the common variables for all devices
    credentials = inventory['common_vars']
    devices_counter = len(inventory['hosts'])
    # get command to execute from CLI
    command = input('nCommand to run: ')
    # loop to keep throwing commands to the same selected inventory
    while command.lower() != 'exit':
        print(f'nExecuting command: {command}n')
        # Start timer variable
        execution_start_timer = time.perf_counter()
        # Loop to repeat the command in all the inventory
        for device in inventory['hosts']:
            # update the device dictionary with the credentials and send command
            device.update(credentials)
            print('*** host: {} - ip: {}'.format(device['hostname'], device['host']))
            # send command and store results
            result = send_command(device, command)
            # show result
            print(f'{result}n')
        # Get and print finishing time
        elapsed_time = time.perf_counter() - execution_start_timer
        print(f"n"{command}" executed in {devices_counter} devices in {elapsed_time:0.2f} seconds.n")
        command = input('Command to run or 'exit': ')
```

_Output example:_

```basic
adrian@adrian:~$ python3 4_bonus.py 
Specify device filter: isp
Matched inventory
*****************
* host: isp-pe - ip: 10.0.12.1
* host: isp-internet - ip: 10.0.14.1
Please confirm (y/n): y
Command to run: show clock
Executing command: show clock
*** host: isp-pe - ip: 10.0.12.1
.14:26:10.489 UTC Thu May 6 2021
*** host: isp-internet - ip: 10.0.14.1
.14:26:25.415 UTC Thu May 6 2021
"show clock" executed in 2 devices in 12.54 seconds.
Command to run or 'exit': 
```

Nice uh!

As you can see, the commands are executed sequentially, and it takes around 6 seconds per command.  

Now this is night and day difference with what we are used to do by opening all the putty sessions, looking for the device, logging in and execute the command... how log it would take? at least 1 o 2 minutes per device and without making any typo error.

If you need to type a command in 10 devices is could take 60 seconds, which  is already amazing in comparison to do it manually.

But wait, we can do even better!

In the next [post](/posts/2021-05-09-how-to-netmiko-full-speed-concurrency-options/), I will add multithreading and multiprocessing examples to execute the tasks concurrently and speed up things.

There is not much else to say, just fill your _**inventory.yml**_ and start throwing commands to your whole infrastructure!

Thanks for reading, have a nice day!
