# Self Driving Nanodegree PID Controller

---

[//]: # (Image References)
[image1]: images/High_Kp_1.0.JPG "High Proportional Constant -1.0"
[image2]: images/Low_Kp_0.1.JPG "Low Proportional Constant -0.1"
[image3]: images/PD_Graph.JPG "PD Controller with Kp=-0.2 and Kd=-3.0"
[image4]: images/PD_controller_Kp-0.1_Kd-0.3.JPG "PD Controller with Kp=-0.1 and Kd=-3.0"

The goal of this project is to adjust steering angle of a car in real-time to keep our Robot/Car within the track using PID Controller.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./pid`. 

---

In order to implement PID controller there are basically three parameters that we have to tune which are as follows:
1. P - Proportional
2. I - Integral
3. D - Derivative

We will be adjusting steering angle of a car by tuning all these three parameters and with crosstrack error (CTE) as an input parameter. The crosstrack error is the lateral distance between the vehicle and the reference trajectory.

**P Controller**

In P Controller, steering angle will be dependent on the Kp constant and on the crosstrack error.

To calculate the steer value: `steering = -Kp * CTE`.

First I tried setting only proportional constant setting other components to zero as `[-1.0, 0.0, 0.0]` and it was working oscillatory. When the car goes right, it steers back to the left and vice versa. The oscillations were like a sin wave on the track.

I learned from classroom session that keeping the Kp constant high results in faster oscillations. Below images can depict the difference observed in the car by tuning Kp through a graph.

![alt text][image1] ![alt text][image2]

So I reduced it to 0.1 and it resulted in much smoother car drive. I passed the first turn with 32mph speed unlike the first try but it crossed the edges after that as we can predict.

**PD Controller**

In PD Controller, the steering angle is not just related to crosstrack error and constant Kp but also to the derivative of the crosstrack error along with Kd constant. That means when the car has turned enough to reduce the crosstrack error it won't shoot the reference line, but it will observe that error is becoming smaller over time and it will counter the steer.

The formula for calculating steer value is: `steering = -Kp * CTE - Kd * diff_CTE`

Refer this screenshot below which shows that how green line (PD Controller) converges to reference line.

![alt text][image3]

Here the derivative of crosstrack error is the difference between cte (crosstrack error) at time t and cte at time t-1 (i.e. previous error).

I set my constant Kd at 1.0 initially and it was able to complete the whole track without falling off the edge. Although it was oscillating much more when there are turns on the track and eventually it shoots over the lane.

I changed Kd to 3.0 and surprisingly the car was able to go on the track without falling off, unlike prior behavior. Other than a little bit of oscillatory behavior it didn't cross the edges. Please refer below image where the PD Controller is working well on turns also along with straight paths.

![alt text][image4]

**PID Controller**

The I Controller is introduced in order to compensate system bias if any. And it is kept relatively small than Kp and Kd.

I component consists of constant Ki and the sum of all crosstrack errors which can be referred as an integration of cte.

The formula for calculating steer value is: `steering = -Kp * CTE - Kd * diff_CTE - Ki * int_CTE`

I tried setting Ki to 0.001 with `Kp=-0.1` and `Kd=-3.0` respectively. But it didn't work well and failed within seconds. Then I tried much lesser constant -0.0001 and it was working well. Still, the car was driving on the boundaries when the complex turns comes up.

## Fine Turning

It turns out that even after tuning Ki to from 0.001 to 0.0001 it didn't make much difference to the problem of driving near boundaries while sharp turning. I figured out the solution that I need to increase my Kp from 0.1. Because low Kp wasn't increasing my steering angle to the desired value when needed because the multiplication factor was very small.

I also had to change my Kd constant in process of increasing my Kp as the car started introducing more oscillations. By increasing Kd to 6.0 oscillations were reduced considerably. And I kept my Ki to really small value as 0.0009.

That makes my final values of constants as `[-0.22, -0.0009, -6.0]`. With these parameters car in a simulator is able to drive without going near to edges on a straight path as well as sharp turning.