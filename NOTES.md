
# CNN-Based Autonomous Navigation for Micro Air Vehicles

**Group 8**: Danish Ansari, Damian Bhawan, Patrick Kostelac, David Schep, Manu Singh, Gijs Zijderveld  
**Code**: [https://github.com/TU-DAnsari/paparazzi](https://github.com/TU-DAnsari/paparazzi)  
**Delft University of Technology**  
April 3, 2024

## Abstract

The goal of the MAV course's (AE4317) project was to cover as much distance as possible with the Parrot Bebop Drone (Zoë, our drone's name) in the CyberZoo, while avoiding random obstacles. For obstacle avoidance, various techniques such as optic flow, mapping, edge detection and monocular depth estimation were explored. The final chosen approach contains a control strategy based on a convolutional neural network model. This CNN model uses depth estimation for labelling a self captured dataset. The CNN model predicts the presence of the obstacle in three segments of the image, with the drone being controlled according to these probabilities while trying to maximise continuous flight. Using this approach, our drone covered a distance of 130.3 metres in the CyberZoo in the allotted 10 minutes.

## Introduction

Many different types of sensors can be used with the purpose of machine perception to aid a Micro Air Vehicle (MAV) in autonomous flight. The problem is that the sensors that provide high-fidelity information about the environment are active sensors, e.g. LIDAR, which increases cost, weight and energy consumption, ultimately leading to shorter flight time. Onboard video cameras are passive sensors that are relatively cheap and light, which make them a more suitable option. This project aims to make use of the onboard mono-camera on a commercially available drone to develop a vision-based obstacle avoidance algorithm.

## Explored Approaches

This section goes over some of the approaches that were initially explored for obstacle avoidance and path planning but were ultimately left out due to various reasons.

### Orange Avoidance

Since a part of the obstacles were guaranteed to be orange, the first implementation of obstacle avoidance was made specifically to avoid orange poles while trying to maximise continuous flight. By calculating the fraction of orange pixels in different regions of the camera images, the weighted difference between the left regions and right regions could be used to set the yaw rate. A frontal obstacle is detected when the fraction of orange pixels exceeds a certain threshold, which results in stopping and turning to search for a safe heading.

### Optic Flow

By calculating optical flow vectors between consecutive camera images, the magnitude of the optical flow vectors can be used to determine how crowded different areas of an image are with obstacles. Using the difference between the magnitude of optical flow between the left side and right side of the image, the heading rate of the drone can be controlled to move away from the obstacles.

### Edge Detection

The edge detection feature was already implemented in one of the paparazzi files, which means that it only had to be implemented in the `orange_avoider` module. The strategy we planned for this was quite simple. The edge detection feature outputs a grayscale image that depicts edges in white. Once we get the edges, the image would then be split up into five segments, and based on the sum of the intensity of the pixels in each segment of the image, the drone would choose its heading. The drone would then turn in the direction where the least amount of edges through the sum of pixel intensities are.

### Monocular Depth Perception

Monocular Depth Estimation is the task of estimating the depth value (distance relative to the camera) of each pixel given a single (monocular) RGB image. For this, a general pre-trained monocular depth perception CNN model was used. The concept and code were taken from Boosting Light-Weight Depth Estimation Via Knowledge Distillation. This paper proposed a lightweight network that can accurately estimate depth maps using minimal computing resources.

### Mapping

The drone maps the area, selects an obstacle-free path, and updates its map based on sensor data. A 2D map is used due to computational limits. Map cells are categorized into three groups: empty, unknown, and obstacle cells. Unknown cells can have both empty and obstacle cells. The updating process includes two methods: First, utilizing the drone's precise location obtained from the OptiTrack system in the CyberZoo as a periodic update to mark safe areas on the map. Secondly, responding to alerts from the local path planner indicates the need to deviate from the global path due to detected obstacles. In such instances, the drone adjusts its global path by removing waypoints and marking cells between the drone and the waypoint as obstacle cells.

## Chosen Approach

### CNN architecture

We used the MobileNetV3 model which is specially designed for computer vision applications in devices with limited computational resources. MobileNetV3 incorporates efficient building blocks such as depthwise separable convolutions and linear bottlenecks to reduce the computational cost while maintaining high accuracy.

#### MobileNetV3 Configuration

MobileNetV3 comes with 2 configurations, `small` and `large`. These configurations define the specifics of the MobileNetV3 blocks that are stacked on top of each other. The 3 most important configuration values are shown in the following tables. The kernel size defines the convolution kernel size used in that block. The Expansion size defines how much the block will expand and then project back. And the channels define the number of channels of the block.

**Large Configuration**

| Kernel Size | Expansion Size | Channels |
|-------------|----------------|----------|
| 3           | 1              | 16       |
| 3           | 4              | 24       |
| 3           | 3              | 24       |
| 5           | 3              | 40       |
| 5           | 3              | 40       |
| 5           | 3              | 40       |
| 3           | 6              | 80       |
| 3           | 2.5            | 80       |
| 3           | 2.3            | 80       |
| 3           | 2.3            | 80       |
| 3           | 6              | 112      |
| 3           | 6              | 112      |
| 5           | 6              | 160      |
| 5           | 6              | 160      |
| 5           | 6              | 160      |

**Small Configuration**

| Kernel Size | Expansion Size | Channels |
|-------------|----------------|----------|
| 3           | 1              | 16       |
| 3           | 4.5            | 24       |
| 3           | 3.67           | 24       |
| 5           | 4              | 40       |
| 5           | 6              | 40       |
| 5           | 6              | 40       |
| 5           | 3              | 48       |
| 5           | 3              | 48       |
| 5           | 6              | 96       |
| 5           | 6              | 96       |
| 5           | 6              | 96       |

**Tiny Configuration (our)**

| Kernel Size | Expansion Size | Channels |
|-------------|----------------|----------|
| 3           | 1              | 8        |
| 3           | 1              | 16       |
| 3           | 4.5            | 24       |
| 3           | 3.67           | 24       |
| 5           | 3              | 40       |
| 5           | 3              | 40       |

#### Dataset Creation

We collected data by flying the drone around and recording 8000 images. These images represented the various obstacles and backgrounds. After collecting the dataset, the RGB images were transformed into depth images using a large Deep Learning model. This model can be seen as the teacher model. DepthAnything was chosen for its state-of-the-art performance.

We segment the RGB images and corresponding depth images into three segments by converting the original images $520 \times 240$ pixels into 3 segments of $170 \times 170$ after removing some pixels from top and bottom. The segments are then downscaled to $85 \times 85$. Then we used these segmented depth images for labelling our dataset. We chose to create labels based on the presence of obstacles nearby. For this, we used a histogram filled with the depth estimation per pixel, ranging from 0 to 255.

We are only interested in the rightmost bin, as this includes all the pixels that are estimated to be close to the drone. If that frequency crosses the threshold of 1500, then we label it as True for a nearby obstacle.

The resulting dataset is imbalanced, the final dataset has 1.9 times more negative samples than positive samples. Since, the task is binary classification, we will have to consider this imbalance while training.

#### Model Training

We used the custom MobileNetV3 tiny model as described in section 3.3.1. For training, we used Adam Optimiser with a learning rate of 0.001 in 8 batches and 40 epochs. We used Binary Cross Entropy as a loss function. This loss function can handle class disparity effectively. The code for the dataset creation, model architecture, model training and converting to C, is all available on our [GitHub repo](http://github.com/DavidEncrypted/bebop_cnn_obstacle_detection).

### Control Strategy

As the goal of the competition was to cover as much distance as possible while avoiding obstacles, maximising continuous flight time is of utmost importance. In the provided `orange_avoider` module, the drone would stop and turn when it detected an obstacle. This resulted in a large portion of time being spent hovering in place.

An optimal control scheme would allow for continuous control, which means the drone should fly forward continuously while the heading rate is used to steer away from obstacles. This cuts down on the amount of time the drone is not covering any distance. To allow for this direct control over linear and angular velocities, the drone is controlled using the guided mode rather than the navigation move.

Using the output of the neural network, continuous control can be implemented by using the difference in probabilities between the left and right regions to apply a heading rate that steers the drone away from the region with the higher probability. With a sufficiently low forward velocity, the CNN can accurately provide probabilities for the presence of obstacles within these different regions. To mitigate the effects of noisy outputs from CNN, the Exponentially Weighted Moving Average (EWMA) was taken over the last 5 outputs for each region of the image.

Additionally, to counteract forward drift when applying a heading-rate, an additional inward velocity in the y-direction of the drone is applied to steer into the bend.

When continuous flight is no longer feasible, i.e. there is a probability of there being an obstacle in the middle of the camera images above a certain threshold, the drone stops and turns to find a safe heading direction. When a frontal obstacle is encountered, the current position $p_o$ of the drone is stored. To avoid collisions with frontal obstacles, a backward velocity that is proportional to the distance between the drone's current position and the saved position $p_o$, is applied. Moreover, this backward velocity ensures that there is enough space between the drone and the frontal obstacle to allow for a turn in place. The position $p_o$ is reset when a new frontal obstacle is detected.

To ensure that the drone stays within the bounds of the CyberZoo, the OptiTrack motion capture system was used to keep track of the drone's xy-position. When the drone reaches the edge, it stops and searches for a safe heading direction. Additionally, when the drone approaches the edge of the CyberZoo, the forward velocity of the drone is scaled down to ensure the drone does not drift outward when stopping at the edge.

## Testing

We tested the code developed for autonomous navigation first in the Gazebo simulator and then in the CyberZoo. In this section, the test phases will be discussed in more detail.

### Simulation

Testing in simulation first is required to save time and resources due to the limited allocated time to test in the CyberZoo. Different obstacle detection approaches were tested in simulation, and those with subpar performance were abandoned. Testing in simulation also helped in bringing forward some software issues which could have been over-sighted if directly tested in CyberZoo. Some issues, however, need real-world testing. First and foremost, the CNN used to detect obstacles could not be tested in simulation, as the obstacles appearing in simulation appear different in real life. One problem that was detected in simulation testing is the potential danger of continuous turning when the drone is close to the edge. In such cases, the drone would enter the close-to-edge case, which turned the drone towards the centre of the CyberZoo. In this case, however, the drone did not perform any obstacle detection, which meant that the drone would crash into nearby obstacles when turning inwards. As this was realized very late, the solution was to remove the continuous turning and stop the drone when close to an edge.

### CyberZoo

Every week during the course, the drone could be tested in real life. Every technique described in sections "Chosen Approach" and "Explored Approaches", which could also be successfully implemented in simulation, was then tested in the CyberZoo. It was important to test in the CyberZoo to refine our control strategy. The control strategy, as described in the "Control Strategy" section, had to be adjusted after every test day in the CyberZoo. We started with the basic control strategy and fine-tuned it by doing the gain tuning, as gains in the simulation model were not enough for the CyberZoo.

Testing in the CyberZoo also helped us in improving our CNN model. The more data is feed to a CNN, the better the CNN will do its job. Data was recorded while flying around and testing, but the best data was gathered while holding the drone in our hands while walking in the CyberZoo. Different obstacles could then be scanned more thoroughly. During the testing of the CNN, it was then important to fly the drone at the same height as the trained data was recorded.

## Results

We got >80% accuracy after training several CNN models and this accuracy was on real-world data. However, since there is a class imbalance, accuracy is not the desired metric for choosing the model. We trained multiple models with different architectures and hyper-parameters and the best model was chosen based on a confusion matrix. We desired the confusion matrix with more false positives than false negatives for the obstacle detection task. The deployed model (v8 accuracy on the test images is 86.157% and the confusion matrix is closest to what we desired:

```
|                   | Predicted Positive | Predicted Negative |
|-------------------|--------------------|--------------------|
| Actual Positive   | 1235               | 306                |
| Actual Negative   | 296                | 2512               |
```

In the competition, our drone covered a distance of 130.3 meters in the CyberZoo. There was an operator error in keeping the drone at an altitude different from the one where the CNN model was trained. This though did not affect the initial phase of the flight where there were fewer obstacles, but its performance deprecated soon when more obstacles were introduced. Once we corrected the altitude, the drone performance was increased and the drone was able to avoid obstacles and navigate smoothly. There was an issue of battery dying and slower prediction for some obstacles that made us lose some time.

## Discussion

Though we chose accuracy as our main metric along with the confusion matrix, we need to look for other metrics such as F1 score which is better at class imbalances and Recall (sensitivity) which considers False Negative as a higher concern. Our CNN model was trained between 40 to 80 epochs due to time constraints. However, we could try with a larger number of epochs to get better results. Moreover, tiny configuration CNN was designed empirically with little data. More data could lead to better performance. Furthermore, we did not do hyper-parameter optimisation which could give us parameters for best performance. Our learning rate was kept once but we can also employ learning rate scheduling. In the future, we would also like to add other explored approaches such as orange avoidance and edge detection to the CNN model to make obstacle detection and avoidance more robust.


















# RO47005 Planning and Decision Making Final Project

**Authors: David Schep (5643384), Dielof van Loon (5346894), Tatsuki Fujioka (5849837)**

## Introduction

In this project, we focus on motion planning for a quadrotor in the presence of static obstacles using state-of-the-art methods, namely the Rapidly-exploring Random Tree (RRT) and the kinodynamic version, RRT-u. These methods are sampling-based incremental global planning techniques that generate paths in environments with randomly generated convex hulls as obstacles. The goal is to compare the performance of RRT and RRT-u algorithms using metrics such as path planning time, traveling time, and traveled distance.

RRT generates a holonomic global path to a goal, whereas RRT-u provides a non-holonomic global path that respects the velocity and acceleration bounds of the drone. This project leverages the RRT-u algorithm proposed by Urban Eriksson, where edges and nodes of a tree are expanded based on a utility function considering the traveling time of trajectories under maximum velocity and acceleration cases.

For the simulation environment, we utilize [gym-pybullet-drones](https://github.com/utiasDSL/gym-pybullet-drones), a Python-based simulator compatible with the gymnasium API, facilitating the development and testing of our motion planners.

## Robot Model

![Diagram of quadrotor](figures/quadrotor.png)

*Figure 1: Diagram of the quadrotor used in simulations.*

The quadrotor model consists of five rigid bodies: the main body and four rotors. The dynamics are simplified by assuming a diagonal inertia tensor due to symmetry, negligible propeller inertia, and ignoring aerodynamic drag. The translational and rotational dynamics are derived using Newton's second law and Euler's rotation equations, respectively, and linearized around hovering attitude for simplicity in control applications.

The workspace and configuration space for the quadrotor are considered as $\mathbb{R}^3$ and $\mathbb{R}^3 \times SO(3)$, respectively. For motion planning, we assume the quadrotor as a point mass to simplify the complexity, resulting in a configuration space of $\mathbb{R}^3$.

## Motion Planning

We implemented both RRT and RRT-u algorithms, where the key difference lies in the connection strategy to the tree. RRT uses a simple straight-line steering function, while RRT-u employs a minimum-time trajectory steering function respecting dynamic constraints. Both planners perform collision checks using hyperplane equations of convex hulls.

### RRT Algorithm

The RRT algorithm expands the tree by randomly sampling points in the configuration space, connecting these through straight-line paths if they do not collide with obstacles. The process attempts to connect directly to the goal with a small probability in each iteration, ensuring that the path converges towards the goal over time.

### RRT-u Algorithm

RRT-u improves upon RRT by considering the dynamic constraints of the quadrotor. It calculates a cost for each potential new connection based on the estimated travel time, which considers the drone's velocity and acceleration. This approach allows RRT-u to generate smoother and more dynamically feasible paths compared to RRT.

## Results

We conducted simulations in environments with 30 randomly placed convex hulls, comparing the performance of RRT and RRT-u across 100 trials. The results, summarized below, show that RRT-u generally outperforms RRT, especially in terms of goal achievement and travel time, due to its smoother trajectory planning.

| Metric             | RRT   | RRT-u |
|--------------------|-------|-------|
| Goal Found (%)     | 85    | 98    |
| Goal Reached (%)   | 94.1  | 93.8  |
| Travel time (s)    | 4.897 | 3.108 |
| Error X (m)        | 70.821| 84.305|
| Error Y (m)        | 70.709| 84.055|
| Error Z (m)        | 10.501| 9.580 |
| Path distance (m)  | 7.254 | 6.758 |

*Table 1: Average results from 100 simulation trials.*

![Comparison of RRT and RRT-u](figures/comparison.png)

*Figure 2: Comparison of RRT and RRT-u planning trees and paths in a sample environment.*

## Discussion

The comparison of RRT and RRT-u highlights the advantages of considering dynamic constraints in motion planning. RRT-u's ability to plan smoother trajectories that respect the drone's dynamic limits results in more efficient and reliable path planning. However, both algorithms still have room for improvement, particularly in optimizing the path length and ensuring robustness across different environments.

## Future Work

Future enhancements could include integrating more advanced control strategies, such as Model Predictive Control (MPC), to improve the tracking performance of the planned paths. Additionally, exploring alternative sampling strategies or cost functions might yield further improvements in planning efficiency and success rates.

## Video Demonstration

**TODO: Insert video of a simulated drone flight demonstrating the motion planning capabilities.**

---

This project showcases the application of advanced motion planning algorithms in challenging environments, highlighting the importance of considering dynamic constraints for aerial robots.
