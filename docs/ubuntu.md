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
if you would like a bulk update on all CPU's to have the minimum scale = maximum scale
```
for x in /sys/devices/system/cpu/cpu*/cpufreq/;do  echo 3400000 > $x/scaling_min_freq; done
```