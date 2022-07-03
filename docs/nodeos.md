## nodeos

> nodeos is the core service daemon that runs on every EOSIO node. It can be configured to process smart contracts, validate transactions, produce blocks containing valid transactions, and confirm blocks to record them on the blockchain.

Depending on the type of system (whether it is baremetal or cloud) you have configured to run your state history, you might want to isolate the cores to prevent the operating system from utilizing them for certain system processes.

Linux provides the capability to allocate certain process ID's (identifier used to uniquely identify a process) to specific CPU cores. The benefit here is the ability to seperate CPU bound applications and have a specific affinity set to get most out of the core frequencies. 

## Scenarios

- You have to run one of Atomic, History or Hyperion API with a SHIP instance (which is risky, but in terms of budgeting preferences), but would like the chain data to be syncronized in a performent manner, while the API requires as much cores as it is entitled too.

To enable process/core isolation, you need to add the following configuration to your Kernel Loader
> **_NOTE:_** In the example, the intention was to isolate cores 7,9,11. Total amount of CPU's = 12
 
> **_NOTE:_** cpu id's start at 0

```
GRUB_CMDLINE_LINUX_DEFAULT="isolcpus=8,10,12"
```
Update your grub configuration and reboot your system once
```
update-grub && reboot
```
If you would like to test your recent changes, you can install a stress utility and launch a stress test against all of the cores to ensure the isolated cores are not used since they should be unavailable unless processes has been delegated to those
```
apt install -y stress
```
To perform the test against all cpu's
```
stress -c 12
```
To schedule a process on a particular core
```
taskset -cp <isolated core id> <process id/id's>
```
Another way to optimize the nodeos processes would be to set the scheduling parameters to type (SCHED_BATCH) which is designed for non-interactive, CPU-bound applications. It uses longer timeslices (short time frame that gets assigned to process for CPU execution). These processes are normally scheduled with lower priority, but may result in considerable speed improvements.

To schedule a process with SCHED_BATCH parameter
```
schedtool -B <process id/id's>
```

## nodeos startup script

>By this time you should be aware that nodes should execute as a daemon process to prevent the need of an interactive session during the lifetime of the process. If your session ends without gracefull termination, your state history will be out-of-sync, and you will need to replay several sets of data which is time consuming, as well put the dependent systems at risk as they should be level with the headblock

When launching nodeos, there will be 3 process id's which will be registered. Ideally, you would like all 3 process id's allocated to seperate cores which is isolated from system processes (benefits explained above)
You can add the following to your startup script to allocate the nodeos pids automatically.
> **_NOTE:_** (This is still based of the example of having isolated cores 7,9,11). This will schedule the nodes pids each on a seperate isolated CPU and schedule it as a SCHED_BATCH service
```
PIDS=`pidof nodeos`
arr=($PIDS)
NODEPROC1=${arr[0]}
NODEPROC2=${arr[1]}
NODEPROC3=${arr[2]}
echo $NODEPROC1
echo $NODEPROC2
echo $NODEPROC3
taskset -cp 7 $NODEPROC1 && schedtool -B $NODEPROC1
taskset -cp 9 $NODEPROC2 && schedtool -B $NODEPROC2
taskset -cp 11 $NODEPROC3 && schedtool -B $NODEPROC3
```

## Set the performance governer (baremetal instances)

>The CPUfreq governor "userspace" allows the user, or any userspace program running with UID "root", to set the CPU to a specific frequency by making a sysfs file "scaling_setspeed" available in the CPU-device directory

By default, service providers will not have their performance governer set to "performance mode" to save on power and costs + extend the lifetime of peripherals. This is a waste and you should be utilizing every core at maximum capacity as you see fit.

Governor summary
- on demand: Expand or reduce resource consumption based on demand.
- Coservative: It is a profile by which you try to keep the level of spending at the basic levels.
- Performance: It is the most devouring of resources since it makes the system available to the tasks trying to give the maximum possible performance in everything.
- It is the most resource-saving profile, reducing energy and system resource consumption to a minimum.

To have a collective view of the governer set on each core, you can issue the following command
```
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_gov*
```
To set each core profile to performance, you can issue the following
```
for x in /sys/devices/system/cpu/cpu*/cpufreq/;do echo performance > $x/scaling_governor; done
```
If you would like certain cores on powersave and some on performance
```
for x in /sys/devices/system/cpu/cpu[0-6]/cpufreq/;do echo performance > $x/scaling_governor; done
for x in /sys/devices/system/cpu/cpu[7-11]/cpufreq/;do echo powersave > $x/scaling_governor; done
```

## Scaling maximum frequency

>The majority of modern processors are capable of operating in a number of different clock frequency and voltage configurations, often referred to as Operating Performance Points or P-states (in ACPI terminology). As a rule, the higher the clock frequency and the higher the voltage, the more instructions can be retired by the CPU over a unit of time, but also the higher the clock frequency and the higher the voltage, the more energy is consumed over a unit of time (or the more power is drawn) by the CPU in the given P-state.

>When attached to a policy object, this governor causes the highest frequency, within the scaling_max_freq policy limit, to be requested for that policy. The request is made once at that time the governor for the policy is set to performance and whenever the scaling_max_freq or scaling_min_freq policy limits change after that.

To find your maximum frequency for each CPU (remember that this could differ between cores), issue the following
```
cpufreq-info
```
In particular, we focus on CPU 31 which has the following scaling policy set
```
analyzing CPU 31:
  driver: acpi-cpufreq
  CPUs which run at the same hardware frequency: 31
  CPUs which need to have their frequency coordinated by software: 31
  maximum transition latency: 4294.55 ms.
  hardware limits: 2.20 GHz - 3.40 GHz
  available frequency steps: 3.40 GHz, 2.80 GHz, 2.20 GHz
  available cpufreq governors: conservative, ondemand, userspace, powersave, performance, schedutil
  current policy: frequency should be within 2.20 GHz and 3.40 GHz.
                  The governor "performance" may decide which speed to use
                  within this range.
  current CPU frequency is 3.40 GHz (asserted by call to hardware).
  cpufreq stats: 3.40 GHz:5.94%, 2.80 GHz:0.37%, 2.20 GHz:93.68%  (685562)
```
You should now have a list of CPU's with the maximum limit and you will be able to adjust the policy scale frequencies to maximum
> **_NOTE:_** In this example, I am scaling CPU 31 to allow policy frequency between maximum states
```
cpufreq-set -c 31 -r -g performance --min 3400000 --max 3400000
Result => 
analyzing CPU 31:
  driver: acpi-cpufreq
  CPUs which run at the same hardware frequency: 31
  CPUs which need to have their frequency coordinated by software: 31
  maximum transition latency: 4294.55 ms.
  hardware limits: 2.20 GHz - 3.40 GHz
  available frequency steps: 3.40 GHz, 2.80 GHz, 2.20 GHz
  available cpufreq governors: conservative, ondemand, userspace, powersave, performance, schedutil
  current policy: frequency should be within 3.40 GHz and 3.40 GHz.
                  The governor "performance" may decide which speed to use
                  within this range.
  current CPU frequency is 3.40 GHz (asserted by call to hardware).
  cpufreq stats: 3.40 GHz:5.95%, 2.80 GHz:0.37%, 2.20 GHz:93.68%  (685562)
```

