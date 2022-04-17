# Drone Control Flight Software
This repo contains motion planning and waypoint following flight software for the CrazyFlie 2.1 minidrone. I developed these scripts to learn, practice, and research drone flight controls and embedded programming. Specifically, this repo features four unique scripts -- two scripts for testing drone hardware and two scripts for drone motion control. The drone scripts are briefly described below:

- `drone-motion-controller.py` commands the drone to autonomously hover and glide around a virtual box.   
- `autonomous-waypoint-follower.py` controls the drone along a pre-generated set of waypoints that resembles a figure 8 pattern.
- `drone-motor-test.py` use this script to test drone motors and the connection between the drone and local machine.
- `logging-connecting-test.py` tests the logging methods provided by the cflib library.

# Commonly Used Drone Methods
The `cflib` library provides a high-level framework with which to control the CrazyFlie drone. This Python-based library used for two-way communication, real-time parameter logging, and motor control. Below, I highlight the most important methods that I commonly use within the four scripts.

## Connecting to the Drone

Before sending any commands, your local system locate and connect to a nearby Crazyflie drone. The code below calls the `cflib` method that will search for a nearby activated drone.

``` python
    cflib.crtp.init_drivers()
    available = cflib.crtp.scan_interfaces()
    for i in available:
        print "Interface with URI [%s] found and name/comment [%s]" % (i[0], i[1])
```

A `Crazyflie()` object is then intialized and used establish a connection to the nearby quadcopter.

``` python
    crazyflie = Crazyflie()
    crazyflie.connected.add_callback(crazyflie_connected)
    crazyflie.open_link("radio://0/10/250K")
```

Similarly the link between your local machine and the drone can be closed with the following line:

``` python
    crazyflie.close_link()
```

## Parameter Logging

Logging flight parameters is a powerful tool used control and debug the quadcopter. Futhermore, logging gives you access to the states of the drone, such as position, altitude, and more. Below is a basic example of how the `SyncLogger` class can be used to start and stop logging. Note that to start the logging process, the local machine must first be connected to the CrazyFlie.

``` python
    # Connect to a Crazyflie
    with SyncCrazyflie(uri) as scf:
        # Create a log configuration
        log_conf = LogConfig(name='myConf', period_in_ms=200)
        log_conf.add_variable('stateEstimateZ.vx', 'int16_t')

        # Start logging
        with SyncLogger(scf, log_conf) as logger:
            # Iterate the logger to get the values
            count = 0
            for log_entry in logger:
                print(log_entry)
                # Do useful stuff
                count += 1
                if (count > 10):
                    # The logging will continue until you exit the loop
                    break
            # When leaving this "with" section, the logging is automatically stopped
        # When leaving this "with" section, the connection is automatically closed
```

## Motion Control

The `cflib` library simplifies motion controls by providing a high-level framework with which to control the drone. Functions that allow the drone to move vertically and horizontally have been provided. However, note that these functions do not apply closed-loop control; these drone will drift if these high-level commands are used without feedback. Below is an example of a simple flight command.

``` python
    with SyncCrazyflie(URI) as scf:
        # We take off when the commander is created
        with MotionCommander(scf) as mc:
            # Move one meter forward
            mc.forward(1)
            # Move one meter back
            mc.back(1)
            # The Crazyflie lands when leaving this "with" section
        # When leaving this "with" section, the connection is automatically closed
```

