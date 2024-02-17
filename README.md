# PV Optimizer

This integration allows you to maximize the self-consumption of the energy production of your solar panels. It allows you to control large household appliances, such as dishwashers or washing machines with simple switches, but also more complex devices to manage, such as swimming pool filtration or heat pump control through applications. dedicated communicating systems requiring multiparametric control mechanisms.

It is written in python under appdaemon (Home Assistant) and should be easily accessible to those who have some knowledge of programming. This should make it easy to modify to improve it and adapt it to your own needs.
# Features
• Taking into account subscriptions: Tempo, Off-peak Hours, Base

• Taking into account the French resale price EDF OA

• Calculation of the device trigger threshold based on the off-peak hour rate and the resale price

• Automatic programming of devices that could not be started during the day during off-peak hours at night.

• Can work with a hot water tank router, if it is possible to recover the instantaneous power delivered to the latter. The addition of a router is even recommended.

• Allows you to control complex systems, such as swimming pool filtration or heat pumps, through separate applications which communicate with the optimizer.
# Prerequisites
- Home Assistant appdaemon add-on installed
- Solar power sensor in Watt available
- It is measured using a current clamp connected to the house power cable. This measure can be positive in the case of withdrawal from the network or negative in the case of export. If you have an ECS router, you must be able to have the power sent to the tank, in order to have the exact available solar power
  P available = P export + P ballon.
- Color of the day for French tempo subscriptions. You can use the integration without this kind of subscription leave “” for the sensor name
- You can use the RTE API for example. https://www.api-couleur-tempo.fr/api.
# Functioning
Each device is described in the application by:
- his name,
- his power,
- its operating time,
- the device switch sensor,
- the off-peak start-up time if necessary.

The program collects the available solar power every minute. 
It shuts down devices that have reached their scheduled run time and attempts to turn on waiting devices based on available power.
The device is started from a power threshold, which depends on the operating cost compared to that of off-peak hours.

The calculation of the power threshold necessary for startup is done according to the formula:
P threshold = P device \* (1-(HC price – Buyback price) / HP price).
This mode allows you to lower the starting power and see if it is better to start during the day or during off-peak hours.

In the event of limited solar production due to cloud cover, the system schedules all devices that could not be started during off-peak hours at night at the times defined in the configuration file.

When a device is started, it completes its entire cycle, even if the available solar power decreases. Indeed, for example, it does not make sense to stop a dishwasher or washing machine in the middle of a cycle.

For those who have a Tempo subscription, the optimizer does not work on red days in order to avoid significant expenses linked to reductions in solar production.

# Installation
1. Install appdaemon add-on from settings / add-ons if not already done.
2. Download the repository[https://github.com/loudemer/PVOptimizer.git]
3. Put the PVOptimizer.py and PVOptimiser.yaml files in the addon_configs/a0d7b954_appdaemon/apps directory
4. Put the optimizerentities.yaml file in the /config/ directory of HA or in a subdirectory dedicated to yaml files if you have one.
# Configuration
1\ Add in addon_configs/a0d7b954_appdaemon/appdaemon.yaml file

``` pvoptimizer_log:
      name: PVOptimizerLog
      filename: /homeassistant/log/pvoptimizer.log
      log_generations: 3
      log_size: 100000
```
This allows you to read the application logs in the /config/pvoptimizer\_log file or directly in appdaemon web server : http://<ip\_ha>:5050

2\. Complete the /addon\_configs/a0d7b954\_appdaemon/apps/PVOptimiser.yaml file:

1) name of your sensor which gives the solar energy available in ‘available\_energy:’
1) type of electricity subscription in ‘subscription:’ must be : Tempo, HeuresCreuses(Off-peak Hours), Base
1) journee\_tempo. sensor which gives the color of the tempo day in ‘tempo\_day:’ put two double quotes if you have another type of subscription
1) Kwh prices in Cents or Euros. For an off-peak hours subscription, enter the prices: prix\_bleu\_hc and prix\_bleu\_hp.

3\. Describe each device in ‘my\_devices:’ respecting the indentation as described below. The file is pre-populated with 2 example devices that you can edit.

``` my_devices:
      dishwasher:
        name: dishwasher
        power: 2000
        duration: 180
        switch_entity: switch.dishwasher
        night_time_on: "00:30:00"
      dryer:
        name: dryer
        power: 2500
        duration: 90
        switch_entity: switch.dryer
        night_time_on: "03:00:00"
```
The power is in Watt, the duration in minutes, the start-up time must be None if it is another application such as swimming pool filtration for example.

4\. Reload the yaml configuration in tools/all configuration

5\. Create the device control dashboard or adapt the one given in the repository.
# The Dashboard
As an example, you will find in the repository a basic configuration example yaml file that can be improved.

It contains 4 entities per device:
• The startup request: input\_boolean.device\_request\_1,
• The sensor switch controlling the device switch.<device>,
• The sensor power of the sensor device.<power device>
• The execution duration sensor: input\_text.device\_duration\_1.

The input_boolean.device_request_x and input_text.device_duration_x sensors are predefined in the optimizerentities.yaml file. x for the definition rank of the device.
You will need to add an entry on the dashboard for the sensor input_boolean. enable_solar_optimizer which allows you to deactivate the application if necessary.

# Basic instructions for use
Once the installation is complete, the integration is operational.
This part concerns devices which are only controlled by a switch sensor.

## Activating a device
To request the activation of a device, simply click on the first sensor (Request).
If the available solar energy is enough, the device switch will be activated, the power delivered will be displayed and the execution time will begin to count down and be displayed every minute.
It is possible to force the start of a device, even if the available solar power is not enough, by clicking directly on the switch. The device entities will then be displayed as if the startup had been carried out by the integration.
## Shutting down a device
The device will shut down when the scheduled run time has elapsed. You can force the shutdown by clicking on the switch.

## Off-peak programming
All devices for which a request has not been satisfied during the day due to lack of sufficient energy are automatically reprogrammed at night during off-peak hours.
To do this, you must ensure that you configure start times during off-peak periods.

## Visualizing problems
The integration generates a log file which is stored in the /config/log/pvoptimizer.log file.
It is also possible to have more details by directly calling the appdaemon debug console: http://<ip_homeassistant>:5050

You will then be able to see the start and stop of the integration in main_log, any errors in error_log and the progress of the integration activity in pvoptimizer_log.

# Advanced use
The integration, PVOptimizer makes it possible to control devices that require additional control parameters.

For example, for a swimming pool, the water temperature must be taken into account to determine the duration and sequencing of the daily filtration cycles.
To do this, it is simpler to control the operation of the device in a separate appdaemon application which communicates with PVOptimizer.

The separate application will then request the activation of the device from PVOptimizer by switching input_boolean.device_request_x to true.

PVOptimizer will then give activation permission by toggling input_boolean.start.device_x to true. 
The separate application will then manage the operation of the device and stop it if necessary before the allotted time. It may subsequently request other start-ups if necessary.

Three applications are currently being evaluated:

- Swimming pool filtration treated with active oxygen,
- Air-water heat pump with heated floor
- Stepless electric vehicle charging

They will be published soon in my repository.

# Uninstallation
You must remove both files PVOptimizer.py and PVOptimiser.yaml in the addon\_configs/a0d7b954\_appdaemon/apps/ directory.
Then the optimizerentities.yaml file in the /config/ directory of HA,
Also remove the PVOptimizer dashboard
Restart HA.

# Performance
The integration has been running for 4 months with a 3 KWp installation. For the moment I am at 95% self-consumption. It remains to evaluate the behavior in summer.