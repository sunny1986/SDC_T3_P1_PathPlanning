# CarND-Path-Planning-Project : Documentation


### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. 
You will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. 
The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. 
The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. 
The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. 
Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

## Model for generating waypoints

In the main function the uWebSockets template helps the code get the localization data. The simulator tells us where the car's x, y, s, d, yaw & speed are at (line 231). These are the variables used to generating
the waypoints. Also we get information regarding other vehicles from the sensor fusion data (line 247) which is required to implement the goals of this project like a safe lane change.
At first I started with the helper code provided in the lessons to get the car drive itself. But this did not keep the car in its lane. Hence I used getXY function to generate 50 points for its path
spaced at 0.45 m from each other to stay within speed limits. This also used the car's angle to maintain a tangential trajectory helped by previous path points. This methodology helped in keeping the
vehicle stay in its lane. However this led to several violations like acceleration and decelration limit, collisions, etc.

Next we bring the concept of generating waypoints through creating our path made up of waypoints fitting a polynomial over a set of predefined points. This starts at line 403 where we create a vector
using either the car's current x, y or x, y from previous points. This way we should have 2 points in the vector, one as the car's current coordinates and a point before that or simply the last
two points from previous path. Next in line 451, we add 3 more waypoints spaced at 30m apart from each other starting at the car's current coordinates. Now the vector consists of a total of 5 points
also called as 'anchor points', like in the project walkthrough video. We also shift our points to be referenced in car's local coordinates so that the starting point is at (0,0) of the vehicle 
and the car is moving at and angle 0 degrees to its heading direction.

The 5 anchor points in the vector represent the known waypoints to which we will fit a polynomial to generate a function that passes through all of these waypoints. We have used a spline fitting tool
(http://kluge.in-chemnitz.de/opensource/spline/) in the code to generate this polynomial function. Then we create another vector called next_x_vals, next_y_vals that contains the actual points that
are fed to the simulator to use as the path points.

We start with creating this path planner vector next_x_vals, next_y_vals using all points leftover from previous path (line 485). Next we set a target point which in our case is 30 m away from the 
current car's coordinates. Then we split up this distance into equally spaced parts such that at each waypoint the car is travelling within the speed limits. These equally spaced points are 
calculated as N in our code. (Look at the visual aid in the video). This function now helps us get target y value given the x value which is required for generating the next set of points for our
path planner vectors. This happens from line 505 to 507. For each x point starting from where the car is currently at, its y point is calculated by asking the spline function. The x points gets added
on to the previous x point and y value is calculated at each x point and so on. These x, y values are then added to the path planner vectors next_x_vals, next_y_vals which are fed to the simulator.

## Lane change 

The lane change for this vehicle is made possible by the sensor fusion data that the simulator is providing us. This data consists of information like their location and speed  of all the cars around us.
This helps us derive our behaviour on how far we should stay from a vehicle ahead of us in the lane to avoid collisions or how to do a safe lane change. This starts at line 265 in the code where we find
out if there are cars in our lane. If there are vehicles in our lane then we calculate the distance from the car in front of us. At line 285 if the car in front of us is within 30m then we decide
to either change the lane or slow down if it is not possible to do a safe lane change. Line 285 to 390 we take care of scenarios for lane change from each of the 3 lanes. 

## Results
The overall combination of safe lane changes and generating waypoints helped navigate the vehicle around the track without collisions or any violations of limits to keep the travel comfortable and safe.