---
title: "How To Netmiko - Full Speed (concurrency options)"
date: 2021-05-09
categories: 
  - "uncategorized-en"
tags: 
  - "netdev"
  - "netmiko"
  - "python"
---

In the previous [post](/posts/2021-05-06-how-to-netmiko-5-minutes-to-all-devices/), we tried Netmiko from: the most basic command to one device, to the whole inventory, and as a bonus filtering devices to stay in a loop to keep throwing command, very useful for troubleshooting scenarios.

The timing was already good in comparison to doing it completely manually. Between 6 and 10 seconds per device (in my lab), against long typing minutes.

**Now we are going to try different options for concurrency. In other words multiprocessing, multithreading (sync and async).**

Concurrency is a beautiful and complex topic. I recommend you to read this great article about it. I warn you not to try to understand everything at the first read. Is not easy but in time you will get it. When it gets weird for you, just skip the rest and come back here.

- [Speed Up Your Python Program With Concurrency](https://realpython.com/python-concurrency/)

Also if you are starting with Python, this is one of my favorites books, it's online, with a search bar, it's free, and 100% practical for learning Python for networking, it is a great reference:

- [Python for network engineers](https://pyneng.readthedocs.io/en/latest/)

We will work with the same scripts as the previous labs, but this time one version per type of concurrency it is implementing.

You can find the code repository [here](https://github.com/aegiacometti/netmiko_full_speed).

I've moved the common functions to a [functions.py](https://github.com/aegiacometti/netmiko_full_speed) so we don't add unnecessary visual noise to the scripts below.

The scripts are fully commented so I won't repeat myself.

Ok, let's set a base for comparing the timing execution. From the most basic in sequence. A simple "show clock" for 6 devices.

```javascript
adrian@adrian:~$ python3 4_bonus.py 
Specify device filter: all
Matched inventory
******************
* host: site1-access - ip: 10.100.200.1
* host: site1-core - ip: 10.100.12.1
* host: isp-pe - ip: 10.0.12.1
* host: site2-access - ip: 10.101.23.2
* host: site2-core - ip: 10.0.13.1
* host: isp-internet - ip: 10.0.14.1
Please confirm (y/n): y
Command to run: show clock
Executing command: show clock
*** host: site1-access - ip: 10.100.200.1
13:09:56.729 UTC Sun May 9 2021
*** host: site1-core - ip: 10.100.12.1
*09:36:58.061 UTC Fri Mar 1 2002
*** host: isp-pe - ip: 10.0.12.1
*09:37:03.554 UTC Fri Mar 1 2002
*** host: site2-access - ip: 10.101.23.2
*09:37:06.897 UTC Fri Mar 1 2002
*** host: site2-core - ip: 10.0.13.1
*09:37:13.562 UTC Fri Mar 1 2002
*** host: isp-internet - ip: 10.0.14.1
.13:10:20.385 UTC Sun May 9 2021
"show clock" executed in 6 devices in 29.36 seconds.
```

**Base:** It took **29.36 seconds**.

**1.- Multiprocessing**

In this first case, we will trigger a new process per SHH session, and we will be using a library called **concurrent.futures** for these processes management.

Pay special attention to the variable "**max\_workers=6**" as this is the variable that indicates how many processes to trigger, being each process tied to a CPU. Remember you have a limited number of CPUs depending on your PC hardware settings.

```python
# This example show how to execute a command concurrently in the devices
# Using multiprocessing
# Change import settings
import time
from concurrent.futures import ProcessPoolExecutor, as_completed
from functions import get_devices, send_command
if __name__ == "__main__":
    # Type device filter by IP or hostname. Partial values or full. Optionally 'all'
    device_filter = input('nSpecify device filter or all: ')
    # Load devices from file with the filter and display matching device
    inventory = get_devices(device_filter)
    devices_counter = len(inventory['hosts'])
    # get the common variables for all devices
    credentials = inventory['common_vars']
    # get command to execute from CLI
    command = input('nCommand to run: ')
    # loop to keep throwing commands to the same selected inventory
    while command.lower() != 'exit':
        print(f'nExecuting command: {command}n')
        # Start timer variable
        execution_start_timer = time.perf_counter()
        # loop to run command in context manager. Using 6 as max Processes to start and wait
        with ProcessPoolExecutor(max_workers=6) as executor:
            future_list = []
            for device in inventory['hosts']:
                # update the device dictionary with the credentials and send command
                device.update(credentials)
                # Add the task to the pool of threads and run
                future = executor.submit(send_command, device, command)
                future_list.append(future)
            # force to wait until the future_list has been executed
            for f in as_completed(future_list):
                print(f.result())
        # Get and print finishing time
        elapsed_time = time.perf_counter() - execution_start_timer
        print(f"n"{command}" executed in {devices_counter} devices in {elapsed_time:0.2f} seconds.n")
        # Enter new command
        command = input('Command to run or 'exit': ')
```

Output example for **_multiprocessing_**:

```javascript
adrian@adrian:~$ python3 netmiko_multiprocess.py 
... skipped lines ...
Executing command: show clock
*** host: site1-access - ip: 10.100.200.1
13:16:37.525 UTC Sun May 9 2021
*** host: site2-access - ip: 10.101.23.2
*09:43:37.235 UTC Fri Mar 1 2002
*** host: site1-core - ip: 10.100.12.1
*09:43:36.446 UTC Fri Mar 1 2002
*** host: isp-pe - ip: 10.0.12.1
*09:43:38.478 UTC Fri Mar 1 2002
*** host: site2-core - ip: 10.0.13.1
*09:43:38.918 UTC Fri Mar 1 2002
*** host: isp-internet - ip: 10.0.14.1
.13:16:42.405 UTC Sun May 9 2021
"show clock" executed in 6 devices in 11.30 seconds.
```

With **Multiprocessing** took **11.30 seconds**!

One third of the base time!

**2.- Multithreading**

Now I will explain why I used that _**concurrent.futures**_ library instead of traditional Threading and Process libraries.

The reason is that _**concurrent.futures**_ is a high-level library that implements the other 2 libraries. It makes it trivial to switch from threads to processes.

If you compare the code below you will see that only 1 line has changed !!!

From **_ProcessPoolExecutor_**:

```python
with ProcessPoolExecutor(max_workers=6) as executor:
```

To **_ThreadPoolExecutor_**:

```python
with ThreadPoolExecutor(max_workers=6) as executor:
```

In this case, the variable "**max\_workers=6**" is indicating how many threads to start. If there are more jobs to run it will simply wait for any of these 6 to finish and launch a new one.

In this case of threads you don't have the limitation of CPUs, you could use much more workers, but take into consideration that you could run in memory limitations, so don't go crazy here.

Ok now the full script just for reference:

```python
# This example show how to execute a command concurrently in the devices
# Using multithreads
# Change import settings
import time
from concurrent.futures import ThreadPoolExecutor, as_completed
from functions import get_devices, send_command
if __name__ == "__main__":
    # Type device filter by IP or hostname. Partial values or full. Optionally 'all'
    device_filter = input('nSpecify device filter or all: ')
    # Load devices from file with the filter and display matching device
    inventory = get_devices(device_filter)
    devices_counter = len(inventory['hosts'])
    # get the common variables for all devices
    credentials = inventory['common_vars']
    # get command to execute from CLI
    command = input('nCommand to run: ')
    # loop to keep throwing commands to the same selected inventory
    while command.lower() != 'exit':
        print(f'nExecuting command: {command}n')
        # Start timer variable
        execution_start_timer = time.perf_counter()
        # loop to run command in context manager. Using 6 as max Threads to start and wait
        with ThreadPoolExecutor(max_workers=6) as executor:
            future_list = []
            for device in inventory['hosts']:
                # update the device dictionary with the credentials and send command
                device.update(credentials)
                # Add the task to the pool of threads and run
                future = executor.submit(send_command, device, command)
                future_list.append(future)
            # force to wait until the future_list has been executed
            for f in as_completed(future_list):
                print(f.result())
        # Get and print finishing time
        elapsed_time = time.perf_counter() - execution_start_timer
        print(f"n"{command}" executed in {devices_counter} devices in {elapsed_time:0.2f} seconds.n")
        # Enter new command
        command = input('Command to run or 'exit': ')
```

Output example for _**multithreading**_:

```javascript
adrian@adrian:~$ python3 netmiko_multithreads.py 
... skipped lines ...
Executing command: show clock
*** host: site1-access - ip: 10.100.200.1
13:29:14.777 UTC Sun May 9 2021
*** host: site2-access - ip: 10.101.23.2
*09:56:15.307 UTC Fri Mar 1 2002
*** host: isp-pe - ip: 10.0.12.1
*09:56:15.555 UTC Fri Mar 1 2002
*** host: site1-core - ip: 10.100.12.1
*09:56:14.880 UTC Fri Mar 1 2002
*** host: site2-core - ip: 10.0.13.1
*09:56:16.275 UTC Fri Mar 1 2002
*** host: isp-internet - ip: 10.0.14.1
.13:29:18.617 UTC Sun May 9 2021
"show clock" executed in 6 devices in 10.13 seconds.
```

With Multithreading took 10.13 seconds!

It's ok, is better, but not a lot better.

Generally speaking, is better to use multithreading for IO-based tasks (like an SSH where we are waiting for an external response) and multiprocessing for CPU-based tasks (like calculations).

Remember that multiprocess is tied to your CPU numbers, and even threads are not, in Python there is a limitation of running one thread per CPU.

Also, consider the memory and load difference between using multiprocessing and multithreading.

I would suggest you, now read again the link I gave you at the beginning of this post.

- [Speed Up Your Python Program With Concurrency](https://realpython.com/python-concurrency/)

**3.- Asyncio**

This is another method that is based on _cooperative multitasking_.

In multithreading, the time at each task run is controlled by the Operating System. So they can be interrupted at any moment.

In _**asyncio**_ tasks are controlled by the script and allowed to run as long as they are doing something and not waiting.

This is a great article to understand asyncio:

- [Async IO in Python: A Complete Walkthrough](https://realpython.com/async-io-python/)

For our case it this could be good, but one important consideration is that in order to use asyncio, the libraries that you plan to use in the path have to support asyncio, so that could be a limitation.

In the case of Netmiko, it does not support asyncio, but since there are always good guys around there, someone already make a version supporting asyncio.

The name is [netdev](https://github.com/selfuryon/netdev) and you can find it here. Thanks, Sergey!

So, now let's go to the asyncio/netdev version of the script:

```python
# This example show how to execute a command concurrently in the devices
# Using asyncio and netdev
# Change import settings
import time
from functions import get_devices
import asyncio
import netdev
async def task(dev, cmd):
    """
    Task executor
    :param dev: device info
    :param cmd: command to execute
    :return: -
    """
    # Remove key not supported by netdev
    hostname = dev['hostname']
    del dev['hostname']
    # Use context manager to open and close the SSH session
    async with netdev.create(**dev) as ios:
        # Send command
        output = await ios.send_command(cmd)
    # Re add variable and generate output
    dev['hostname'] = hostname
    print('*** host: {} - ip: {}n{}n'.format(hostname, dev['host'], output.strip()))
async def run(hosts, cred, cmd):
    """
    Generate list of tasks
    :param hosts: device list
    :param cred: credentials
    :param cmd: command to execute
    :return: -
    """
    tasks = []
    for host in hosts:
        host.update(cred)
        tasks.append(task(host, cmd))
    await asyncio.wait(tasks)
if __name__ == "__main__":
    # Type device filter by IP or hostname. Partial values or full. Optionally 'all'
    device_filter = input('nSpecify device filter: ')
    # Load devices from file with the filter and display matching device
    inventory = get_devices(device_filter)
    devices_counter = len(inventory['hosts'])
    # get the common variables for all devices
    credentials = inventory['common_vars']
    # get command to execute from CLI
    command = input('nCommand to run: ')
    # loop to keep throwing commands to the same selected inventory
    while command.lower() != 'exit':
        print(f'nExecuting command: {command}n')
        # Start timer variable
        execution_start_timer = time.perf_counter()
        # get event loop and run it
        loop = asyncio.get_event_loop()
        loop.run_until_complete(run(inventory['hosts'], credentials, command))
        # Get and print finishing time
        elapsed_time = time.perf_counter() - execution_start_timer
        print(f"n"{command}" executed in {devices_counter} devices in {elapsed_time:0.2f} seconds.n")
        # Enter new command
        command = input('Command to run or 'exit': ')
```

Output example for **asyncio**:

```javascript
adrian@adrian:~$ python3 netdev_asyncio.py 
... skipped lines ...
Executing command: show clock
*** host: site1-access - ip: 10.100.200.1
14:15:50.333 UTC Sun May 9 2021
*** host: site1-core - ip: 10.100.12.1
*10:42:48.838 UTC Fri Mar 1 2002
*** host: site2-core - ip: 10.0.13.1
*10:42:50.001 UTC Fri Mar 1 2002
*** host: site2-access - ip: 10.101.23.2
*10:42:48.649 UTC Fri Mar 1 2002
*** host: isp-pe - ip: 10.0.12.1
*10:42:50.901 UTC Fri Mar 1 2002
*** host: isp-internet - ip: 10.0.14.1
.14:15:53.273 UTC Sun May 9 2021
"show clock" executed in 6 devices in 4.87 seconds.
```

With **asyncio** took 4.87 seconds!

Now, this is a time improvement in seconds:

- Sequential 29.36
- Multiprocessing: 11.30
- Multithreading: 10.13
- Asyncio: 4.87

Ok, which one should you use? I would go for multithreading for networking which is an IO-based king of tasks. Then it depends on what kind of concurrency you are dealing with the rest of the script, maybe at some point is only calculations, and is better to use multiprocessing. Asyncio seems very good I would use it if I need max speed.

Thanks for reading.-

Adrián.-
