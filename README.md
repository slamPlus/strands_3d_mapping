scitos_3d_mapping
=================

Tools for building 3D maps and using these maps for navigation and visualization.

Start the system
=================
Start all the nodes in this repository using:

```roslaunch semantic_map_launcher semantic_map.launch```


Data acquisition
=================

To collect sweeps, use the action server from:

To start the action server manually (already launched with `roslaunch semantic_map_launcher semantic_map.launch`):

```rosrun cloud_merge do_sweep.py```

Use:

```rosrun actionlib axclient.py /do_sweep```

This action server takes as input a string, with the following values defined: "complete", "medium", "short", "shortest". Internally the action server from `scitos_ptu` called `ptu_action_server_metric_map.py` is used, so make sure that is running. 

The behavior is the following:
* If sweep type is "complete", the sweep is started with parameters -160 20 160 -30 30 30 -> 51 positions
* If sweep type is "medium", the sweep is started with parameters -160 20 160 -30 30 -30 -> 17 positions
* If sweep type is "short", the sweep is started with parameters -160 40 160 -30 30 -30 -> 9 positions
* If sweep type is "shortest", the sweep is started with parameters -160 60 140 -30 30 -30 -> 6 positions (there might be blank areas with this sweep type, depending on the environment).

Calibrate sweep poses
==========================
Once a number of sweeps of type "complete" have been collected, you can run the calibration routine which will compute the registration transformations for the 51 poses. Afterwards, you can execute sweeps of any type (from the types defined above) and the correct transformations will be loaded so that the sweeps are registered.

To start the action server manually (already launched with `roslaunch semantic_map_launcher semantic_map.launch`):

```rosrun calibrate_sweeps calibrate_sweep_as```

Use:

```rosrun actionlib axclient.py /calibrate_sweeps```

(Here you have to specify the minimum and maximum number of sweeps to use for the optimization. To get good registration results you should have collected > 5 sweeps. Note that only sweeps of type "complete" are used here, all others are ignored). 

Once the calibration has been executed, the parameters are saved in `~/.ros/semanticMap/` from where they are loaded whenever needed. All sweeps recorded up to this point are automatically corrected using the registered sweeps.

Reinitialize the metarooms
============================
After the calibration you can re-initialize the metarooms (in general a good idea, as the registration between the sweeps should be better now that the poses have been calibrated).

```rosservice call /local_metric_map/ClearMetaroomService "waypoint_id: - 'WayPointXYZ' initialize: true"```

Set the argument initialize to `true` and provide all the waypoints for which you want to re-initialize the metarooms in the `waypoint_id` list. 
