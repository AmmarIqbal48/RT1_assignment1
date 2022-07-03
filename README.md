# RT1_assignment1

Python Robotics Simulator
================================

This is a simple, portable robot simulator developed by [Student Robotics](https://studentrobotics.org).
Some of the arenas and the assignment has been modified for the Research Track 1 course. In this simulator the robot will spawn inside of an arena composed of squared tokens of two different colors **Silver** token and **Golden** tocken.

1)**Golden tokens:** Walls of the simulator are made by Golden token.

2)**Silver tokens:** Silver tokens are randomly placed in the simulator which robot has to collect.

![Screenshot 2022-05-31 142304](https://user-images.githubusercontent.com/104999107/171596607-f01fd71a-2c13-4df6-bf28-a47864661a6b.png)



Installing and running
----------------------

The simulator requires a Python 2.7 installation, the [pygame](http://pygame.org/) library, [PyPyBox2D](https://pypi.python.org/pypi/pypybox2d/2.1-r331), and [PyYAML](https://pypi.python.org/pypi/PyYAML/).

Once the dependencies are installed, simply run the `test.py` script to test out the simulator.

## Assignment_1
-----------------------------

To run one or more scripts in the simulator, use `run.py`, passing it the file names. 

I am proposing you three exercises, with an increasing level of difficulty.
The instruction for the three exercises can be found inside the .py files (exercise1.py, exercise2.py, exercise3.py).

When done, you can run the program with:

```bash
$ python run.py assignment.py
```

Robot API
---------

The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api].

### Motors ###

The simulated robot has two motors configured for skid steering, connected to a two-output [Motor Board](https://studentrobotics.org/docs/kit/motor_board). The left motor is connected to output `0` and the right motor to output `1`.

The Motor Board API is identical to [that of the SR API](https://studentrobotics.org/docs/programming/sr/motors/), except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following:

```python
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25
```

### The Grabber ###

The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the `R.grab` method:

```python
success = R.grab()
```

The `R.grab` function returns `True` if a token was successfully picked up, or `False` otherwise. If the robot is already holding a token, it will throw an `AlreadyHoldingSomethingException`.

To drop the token, call the `R.release` method.

Cable-tie flails are not implemented.

### Vision ###

To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The `R.see` method returns a list of all the markers the robot can see, as `Marker` objects. The robot can only see markers which it is facing towards.

Each `Marker` object has the following attributes:

* `info`: a `MarkerInfo` object describing the marker itself. Has the following attributes:
  * `code`: the numeric code of the marker.
  * `marker_type`: the type of object the marker is attached to (either `MARKER_TOKEN_GOLD`, `MARKER_TOKEN_SILVER` or `MARKER_ARENA`).
  * `offset`: offset of the numeric code of the marker from the lowest numbered marker of its type. For example, token number 3 has the code 43, but offset 3.
  * `size`: the size that the marker would be in the real game, for compatibility with the SR API.
* `centre`: the location of the marker in polar coordinates, as a `PolarCoord` object. Has the following attributes:
  * `length`: the distance from the centre of the robot to the object (in metres).
  * `rot_y`: rotation about the Y axis in degrees.
* `dist`: an alias for `centre.length`
* `res`: the value of the `res` parameter of `R.see`, for compatibility with the SR API.
* `rot_y`: an alias for `centre.rot_y`
* `timestamp`: the time at which the marker was seen (when `R.see` was called).

For example, the following code lists all of the markers the robot can see:

```python
markers = R.see()
print "I can see", len(markers), "markers:"

for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} metres away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} metres away".format( m.info.offset, m.dist )
```
Functions Used In The Program
---------

Function **drive** is used to drive the robot in the arena, the speed of all the motors in the robot is kept same so the robot moves in straight line.

**def drive(speed, seconds):**
    
    # Function for setting a linear velocity 
    #arguments for function are speed and seconds(time)
    
    
    R.motors[0].m0.power = speed
    R.motors[0].m1.power = speed
    time.sleep(seconds)
    R.motors[0].m0.power = 0
    R.motors[0].m1.power = 0

Function **turn** is used to turn the robot in the arena, the robot turns exactly around its axis.

**def turn(speed, seconds):**
    
    #Function for setting a angular velocity 
    #arguments for function are speed and seconds(time)
    
    
    R.motors[0].m0.power = speed
    R.motors[0].m1.power = -speed
    time.sleep(seconds)
    R.motors[0].m0.power = 0
    R.motors[0].m1.power = 0

Function **find_silver_token** is used to find the silver tokens in the arena command **R.see** is used to find the closest token and (than the grab function is used to grab the silver token and put behind) we have given particular angel to R.see command so the robot can't see the tokens behind it.

**def find_silver_token():**
    
    #Function to find the closest silver token
    """Returns: dist (float): distance of the closest silver token 
	        rot_y (float): angle between the robot and the silver token """
    
    dist=3
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_SILVER and -70<token.rot_y<70: 
    
     	   #the field of view for detectig silver token is restricted to an angle of -70, 70 degrees. 
     	   #In this way the robot can detect silver token just in front of it. 
     	   #This is useful for avoid the robot detecting silver token behind it.
  
	     dist=token.dist
	     rot_y=token.rot_y
    if dist==3:
	return -1, -1 #(-1 if no silver token is detected)(-1 if no silver token is detected)
    else:
   	return dist, rot_y

Function **find_golden_token** is used to find the golden tokens in the arena, the walls of the arena are made of golden token and to avoid the collision with the walls using R.see command the robot finds the golden token and turns left or right to avoid collision we have given a particular angel so robot can't see behind it. 

**def find_golden_token():**
    
    #Function to find the closest golden token
    """Returns:dist (float): distance of the closest golden token"""
    
    dist=100
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_GOLD and -40<token.rot_y<40: 
    
    #the field of view for detectig gold token is restricted to an angle of -40, 40 degrees. 
    #In this way the robot can detect gold token just in front of it.
    
	     dist=token.dist
    if dist==100:
	return -1 #(-1 if no golden token is detected)
    else:
   	return dist

Function **find_golden_token_left** is used to see golden token which are on the left of the robot and avoid collision with them.

**def find_golden_token_left():**
    
    #Function to compute the closest golden token distace on the left of the robot.

    """Returns:dist (float): distance of the closest golden token on the left of the robot"""
    
    dist=100
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_GOLD and -105<token.rot_y<-75:
        #The (-105, -75) angle span is useful for detecting gold token on the left
            dist=token.dist
    if dist==100:
	return -1 #(-1 if no golden token is detected on the left of the robot)
    else:
   	return dist

Function **find_golden_token_right** is used to see golden token which are on the right of the robot and avoid collision with them.

**def find_golden_token_right():**
    
    #Function to compute the closest golden token distace on the right of the robot.

    """Returns:dist (float): distance of the closest golden token on the right of the robot"""

    dist=100
    for token in R.see():
        if token.dist < dist and token.info.marker_type is MARKER_TOKEN_GOLD and 75<token.rot_y<105:
         #The (75, 105) angle span is useful for detecting gold token on the right
            dist=token.dist
    if dist==100:
	return -1 #(-1 if no golden token is detected on the right of the robot)
    else:
   	return dist

Function **grab_routine** is used to grab the silver token before the silver token is grabbed bt the robot the robot has to adjust its angel and postion with respect to the silver token and than grab the token.

**def grab_routine(rot_silver, dist_silver):**
    
    #Function to control the routine for grabbing silver tokens    
    """Arguments for the function are :rot_silver (float):angle between the robot and silver token
    	                               dist_silver (float): distance silver token"""
    """No returns"""
    
    if dist_silver <= d_th: 
    	print("Silver token Found!")
    	if R.grab():
	    print("Grabbed!")
	    turn(20, 3)
	    drive(15, 0.8)
	    R.release()
	    drive(-15,0.8)
	    turn(-20,3)
    elif -a_th<=rot_silver<=a_th:
	    drive(40, 0.5) 
	    print("Adjust your angle")
    elif rot_silver < -a_th: 
	    print("Move Left a littel...")
	    turn(-5, 0.3)
    elif rot_silver > a_th:
	    print("Move Right a littel...")
	    turn(+5, 0.3)

Function **turn_method** is used to turn the robot correctly in the arena there is threshold given by us that the robot comapres with the closest golden token on the right or left and depending on that throshold distance robot moves left or right.

**def turn_method(left_dist, right_dist, dist_gold):**
	
	#Function that implements the turn decision method
	"""Arguments for the function:left_dist (float):distance from the gloden token closest on the left of the robot
		                      right_dist (float):distance from the gloden token closest on the right of the robot"""
	"""No returns"""	
	
	print("Where's the wall?")	
	if left_dist>1.2*right_dist:
		print("The wall is on the right at a distance of: "+ str(right_dist))
		while dist_gold<gold_th: #until there are no more gold tokens in front of the robot
			dist_gold=find_golden_token()
			turn(-10, 0.1)
			print("I'm turning left")
	elif right_dist>1.2*left_dist:
		print("The wall is on the left at a distance of:  "+ str(left_dist))
		while dist_gold<gold_th:
			dist_gold=find_golden_token()
			turn(10, 0.1)
			print("I'm turning right")
	else:
		drive(15,0.5)
		print("Left and right distances are similar, i'll go straight")

Function **main** is the function where every other function is called the main function is kept in loop so the program runs the map infinite times.

**def main():**
	
	while 1:  
			
			#Updating variables value for every while cycle
			dist_silver, rot_silver = find_silver_token()
			dist_gold=find_golden_token()
			left_dist=find_golden_token_left()
			right_dist=find_golden_token_right()
			#Check if gold token are detected, if no gold token are detected go straight ahead.						
			if (dist_gold>gold_th and dist_silver>silver_th) or (dist_gold>gold_th and dist_silver==-1):
					print("I'll go straight ahead")
					drive(70,0.5)		
			#If gold token are detected, then check where the wall is			
			elif dist_gold<gold_th and dist_gold!=-1:
			#The robot decides where to turn
					turn_method(left_dist, right_dist, dist_gold)					
			if dist_silver<silver_th and dist_silver!=-1: 
			#If any silver token closer than silver_th is detected, the grab routine will start
		    		print("Silver is close")
		    		grab_routine(rot_silver, dist_silver)	


Flow Chart 
---------
Basic stucture of the program is explained in the flow chart.

![Flowchart](https://user-images.githubusercontent.com/104999107/171596016-13eb8ff5-cf4d-4134-b2ed-0439f11bd65e.png)

Vedio  
---------
https://user-images.githubusercontent.com/104999107/177034694-ab7f5f29-265b-4c70-b15d-ea369877eaef.mp4

Conclution And Future Improvements
---------
The robot changes direction when it faces a wall (golden tokens). So when robot is moving in the simulator and putting Silver token behind itself there could be condition when the robot places a token which is very closed to the wall and this will stop robot from collecting the Silver token because the Silver token would be in the threshold we have given to the robot so it dosen't collide with the wall.

[sr-api]: https://studentrobotics.org/docs/programming/sr/
