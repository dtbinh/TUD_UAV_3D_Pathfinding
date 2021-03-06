#3D pathgeneration and pathfollowing for UAV in VREP 
This repository contains python code, that can be ran in combination with the Roboter Simulation Platform VREP. We used this Platform to simulate Pathfinding, Pathsmoothing und Pathfollowing for an UAV in some different areas(one small test area and a bigger area showing 2 buildings of our univesity)

## Requirenments:

Tested in Windows 8.1 and Mac OS with Python 2.7 in Spyder

## Installation

1. Install VREP

2. Clone the repo

3. Start the simulation of an area

4. Run the UAV_main.py to start the mapdatageneration, pathgeneration and pathfollowing algorythm

	###Step 1:
	
	Generates an array with the mapdata for the current scene in VREP
	
	Step 2:
	
	Reads the starting position and the goal from the scene
	
	Step 3:
	
	Finds and interpolates a Path
	
	Step 4:
	
	Starts the pathfollowing
	
	Step 5:
	
	After the goal is reached, it shows the calculated path in comparison to the real path the UAV was flying in 2D.
## Documentation

	###UAV_main.py
		####imports
		vrep.py, for the API-functions 
		UAV_mapgen.py, for the mapdata-functions
		UAV_pathfinding.py, for the pathfinding-functions
		UAV_VREP.py, for the functions to communicate with V-REP
		numpy.py, for some math-functions
        matplotlib.pyplot and mpl_toolkits.mplot3d, for plotting
		####code
		Contains the main-script of our project, here are all other parts combined. First a connection to V-REP is established, 
		then the mapdata is generated or loaded if she already exists. Next step is to read the start and the goal position from V-REP. 
		Thats all information, which is needed for the pathfinding and generation. After the pathgeneration is finished a signal is send to V-REP,
		so the path can be drawn in the simulation by a part in the UAV-LUA-script.
		Thats the point, when the path-following can be started. V-REP is ran in the synchronous-mode, to make sure for every simulation step the 
		information/direction/orientation is updated.
		After the path-following is finished a plot is created where the real path which the UAV was flying is compared with the calculated path in 2D.
		The xy-plane is shown, the calculated path is shown as line, the real path is shown by the points where the UAV was before the next simulation step was executed.
	
	###UAV_VREP.py
		####imports
		vrep.py, for the Connection with the Simulator
		numpy.py, for the arrays and some other mathematical operations
		time.py, for the sleep-function
		math.py, for some math-functions
		pathfollowing.py, for the function which returns the direction the UAV need to move to follow the path
		####functions
			#####getPosition
				######input
				clientID, integer, needed to execute API-functions
				object_name, string, the name of the object, which position you want to get
				######output
				position, array with the 3 coordinates x,y,z in meters
				######code
				First the object handle is returned from V-REP, afterwards its possible to get the position from V-REP.
			#####angle_calculation
				######input
				a and b, arrays(1x3), 3D-vectors
				######output
				angle, float, the angle between the two vectors in radiant, always returns the small angle
				######code
				Simple angle calculation with the scalar-product and the arcus-cosinus using functions from math.py
			#####show_path
				######input
				path, array, contains 1 array for each coordinate, the length of these arrays depends on the length of the path
				clientID, integer, needed to execute API-functions
				######output
				no return value, but this function creates string signals with the coordinate arrays, which are read from the UAV-LUA-script
				######code
				After the path-array is divided, the 3 signals are created.
			#####followPath
				######input
				clientID, integer, needed to execute API-functions
				path, array, contains 1 array for each coordinate, the length of these arrays depends on the length of the path
				goal, tupel, contains the 3 coordinates of the goal-position
				######output
				plot-data, array, contains the position of the UAV in each simulation step
				######code
				Depending on the current position of the UAV and the path a direction for the UAV is calculated and converted into velocities, 
				which are given to the UAV, by creating string signals, that are checked by the UAV-LUA-script. After the calculation is finished 
				the next simulation-step is tiggered. This algorythm is repeated, till the goal-position is reached.
				Afterwards the signals are cleaned up and some plot-data is returned to the main-script.
				
        ###UAV_mapgen.py
            ####imports
				vrep
				sys
				numpy
				time
				math
				interpolate
				deepcopy
				os.path
				cPickle 
	     	####functions
				#####save
	     	        ######input 
						array with mapdata
	                ######output
	                    array with modified mapdata 
	                ######code
						the obstacles are extended, to avoid collisions
				#####mapgen
	     	        ######input 
						scene_name, string, name of the area used as name for the mapdata file
						x,y,z, integer, dimensions of the area, which will be detected
						clinetID, integer, needed to execute API-functions
	                ######output
	                    array with detected mapdata
						a file with the mapdata
	                ######code
						the sensors are moved through the whole area to detect the obstacles
				#####mapgen_fast
	     	        same structure as mapgen, but using more sensors to increase the speed
        
		###UAV_pathfinding.py
            ####imports
				division
				vrep, needed for the Connection with the Simulator
				sys
				numpy, needed for the arrays and some other mathematical operations
				time
				math
				interpolate, needed for the interpolation functions
				collections, needed for the queue       
				heapq, needed for the queue
				random, needed for RRT
				deepcopy
	     	####functions
			There is one main-function "search", which calls other functions, depending on the choosen algorythm. After this part the path is interpolated by using a existing python-libary for it.
		
		###pathfollowing.py
            ####imports
				numpy, needed for the arrays and some other mathematical operations
	     	####functions
				#####findnearest
	     	        ######input 
						position, tupel
						path, array
	                ######output
	                    direction, array
	                ######code
					Depending on the current position and the path a direction is calculated to follow the path.
		
        ###Scene: hexagon_neu.ttt
                #### Abstract: In the scene we have the S311 building which contains the walls and the windows. The goal_new object is the goal which we want the UAV to fly to. You can also move the goal. The UAV script is the main part of the scene. It can control the UAV and also draw the path which we calculated. We learned from our betreuer Raul's script, which will also be discribed on the following. 
                #### UAV code
                        #####1. modify the original quadrotor control to receive Twist commands from ROS.
                        #####2.get the path from python and show the path in v-rep.
                        #####3.We tried to control the quadricopter vertical and horizontal, bzw x,y and z direction. Then we got the error in Alpha, Beta and Rotation, which result in different velocities of the 4 rotors. Then we send the velocities to the rotors and let the quadcopter work properly.  We optimise the parameters to make the rotors work better.
                         
                         
        ###Rauls quadricopter code
                ####The original quadricopter script was given to us by Raul Acuna. The script has the following parts.
                ####code       
                        #####1.ROS initialization
                        #####2.modify the original quadrotor control to receive Twist commands from ROS.
                        #####3.Prepare 2 floating views with the camera views.
                        #####4.Control the quadricopter vertical and horizontal by deciding the motor velocities.
                        #####5.Move the target object.
                
        