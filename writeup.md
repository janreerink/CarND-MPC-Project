# Writeup Model Predictive Control Project
This writeup provides an overview of MCP and the implementation as part of the corresponding Udacity self-driving car project.

## Model state
The model state has the following components: x-position, y-position, orientation, velocity, steering angle and acceleration. To control the state two actuators (steering angle and acceleration) are used to reduce the cross-track and orientation error. 

Updates to the model state are implemented using the functions shown in class. The next position is a function of the time step duration, angle and velocity. The new orientation also requires a vehicle specific input (L, the distance between center of mass and front axle) as well as the steering angle, velocity, time step and original orientation. The updated velocity is given by initial velocity, time step duration and acceleration.

## Setting n, dt
The number of timesteps N and duration between actuations dt were set to 10 and 0.14 respectively. N and dt define the prediction horizon, i.e. the timeframe for which the model makes predictions. A T of several seconds is sufficient for this use-case as the environment can change quickly, making longer predictions less reliable. N determines the length of the vector of variables to be optimized and thus drives computational cost. dt should be small to reduce the discrezitation error, however, decreasing the value also increases computational cost.

## Cost fucntion components
The cost function is implemented as shown in class and has three components: the current vehicle state (cross-track error, orientation error and difference to reference veloity), two variables to minimize actuator input by adding the magnitude of actuator input and two more variables to minimize the difference between subsequent actuator inputs. The two actuator-related components lead to a smoother path from current to target path.
The costs were multiplied by manually tuned factors to weight their relative importance.

## Setting constraints
To optimize we need a vector with N observations for each state element. We initialize the first N elements of this vector to the current state. Subsequent values are set using the update function and the respective previous states.
Bounds have to be set for the optimizer: the steering angle is bounded between -25 and 25 degrees. Acceleration to -1 and 1. 

## Application
The main.cpp uses the solve method of the MPC class using the state and waypoints provided by the simulator as input. The waypoints needs to be transformed into vehicle coordinates. The solver returns actuator inputs and predicted points that are sent back to the simulator.

## Dealing with latency
Latency was finally compensated by predicting the state after latency in the main.cpp and passing the predicted state to the solver, which then finds actuations to optimize correct state.

# Tuning
The weights for the cost functions and the values for N and dt were tuned to keep the vehicle on the road and reduce oscillations. Initially the cost weights were kept at 1 and values for N and dt taken from class (10, 0.5). This did not work, so delta t was decreased. While the car stayed mostly on the road, small values of dt also led to strong oscillations. 
After some tweaking of the weights for cross-track, orentation and actuator input the car stayed on the road without oscillations. To increase reference speed (initially at 60) the time horizon was increased slightly while also increasing the weighting for the acutation-related cost components, which enabled the car to anticipate curves better, at the cost of larger error close to the car. The final reference speed (100) with a slightly increased velocity weight factor is a good trade-off between high speed on relatively straight patches and safety on the rest of the track.
After some feedback from reviewers and changes to the latency compensation the tuning had to be repeated. A lower timestep combined with differently scaled cost weights produced much better results.


[Example video](mpc.mp4)
