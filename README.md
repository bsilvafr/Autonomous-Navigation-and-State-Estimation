# Autonomous-Navigation-and-State-Estimation

## From a Pendulum to EKF and Particle Filters
#### ⚠️ Important Notice

This is a personal project developed independently for learning purposes.
The code reflects my study process on Kalman Filters and Particle Filters.
As an educational project, it is constantly evolving and may contain implementation errors or concepts that were refined over time.

### 🎯 About the Project

This project started as a personal challenge aimed at understanding, in practice, how state estimation algorithms work in the context of navigation, especially when dealing with inertial sensors.

Rather than simply implementing known solutions, the goal was to build intuition progressively — starting from very simple systems and gradually evolving toward more realistic scenarios involving multiple sensors, noise, asynchrony, and non-linear dynamics.

The entire work follows an incremental and evolutionary approach, where each stage introduces new challenges, exposes limitations of the previous model, and motivates the next improvement.

### 🧠 Fundamentals
##### 🔷 Kalman Filter (KF / EKF)

The Kalman Filter is a mathematically grounded approach based on linear algebra and probability theory. It assumes Gaussian noise and operates by continuously balancing two sources of information: the system model (prediction) and sensor measurements (correction).

When the system becomes non-linear, the Extended Kalman Filter (EKF) is used. It relies on local linear approximations (Jacobians) to handle non-linearities. This method is computationally efficient and stable, but highly dependent on model accuracy and underlying assumptions.

#### 🔶 Particle Filter (PF)

The Particle Filter takes a completely different approach. Instead of maintaining a single estimate with covariance, it represents the system state as a distribution of hypotheses (particles).

Its operation can be summarized in three main steps:

* Particles are propagated using a motion model with noise

* Each particle is weighted based on sensor measurements

* A resampling step selects and replicates the most likely hypotheses

This approach does not rely on Gaussian assumptions and handles non-linearities naturally. However, it comes with higher computational cost and strong sensitivity to parameter tuning.

### 🛤️ Incremental Development Journey
#### 🔹 Phase 1: Pendulum — Understanding Covariance

The project begins with a simple pendulum model, used to validate the basic behavior of the Kalman Filter.

Sensor loss was simulated, along with scenarios where the model did not perfectly represent reality (e.g., unknown damping). This revealed a key insight: in the absence of measurements, the filter fully trusts its prediction, causing uncertainty to grow rapidly. When the model is inaccurate, this leads to divergence, which is only corrected when measurements return.

![Pendulum](https://github.com/user-attachments/assets/80f6ceda-06d6-4e75-bed3-1ae5ac3b0e5f)

#### 🔹 Phase 2: 2D Trajectory with Position Measurements

The system was then extended to a 2D motion model with position and velocity states, but only position measurements.

This setup enabled a direct comparison between the Kalman Filter and the Particle Filter. By introducing measurement outliers, it became clear that the Particle Filter is slightly more robust, while the Kalman Filter remains more stable under normal conditions. The impact of increasing noise (sigma) was also analyzed, showing different degradation behaviors.

![KF_1](https://github.com/user-attachments/assets/76d09f1f-dad0-4c40-ae8c-b16ad8c27076)
![PF_1](https://github.com/user-attachments/assets/493437d2-4a23-446f-a476-bbd3f485c752)

#### 🔹 Phase 3: Synchronous Sensors (Position + Velocity)

In this phase, sensors provided both position and velocity measurements synchronously.

This significantly improved system observability and reduced estimation errors. It clearly demonstrated how additional measurements enhance the filter’s ability to correct accumulated errors and stabilize the estimate.

#### 🔹 Phase 4: Asynchronous Sensors (IMU + Radar)

The system was made more realistic by introducing asynchronous sensors: an IMU running at 20 Hz and a radar providing position at 1 Hz.

This introduced challenges related to time handling and prediction updates between measurements. A noticeable position drift appeared due to continuous integration of errors when measurements were sparse.

This stage highlighted two critical aspects:

* proper handling of time steps (dt)

* the role of observability in system stability

![Drift_pos](https://github.com/user-attachments/assets/276854da-5b40-480f-9065-bbc7ce778cf5)

#### 🔹 Phase 5: Acceleration Bias Estimation

To address the observed drift, acceleration bias was introduced into the state, and the IMU model was updated to measure acceleration instead of velocity.

Even with noisy bias estimates, this change significantly improved convergence and eliminated long-term drift.

![No_drift](https://github.com/user-attachments/assets/16863efc-ba1c-46ac-a16f-0ae9feb8f5dc)

#### 🔹 Phase 6: Orientation and Reference Frames

The model was extended to include orientation (θ), requiring coordinate transformations between the body frame (IMU) and the global frame (GNSS).

A major issue emerged: the orientation did not converge. Without direct observation, the filter could not distinguish whether the error came from incorrect orientation or acceleration bias — a classic observability problem.

At this point, the EKF was introduced to handle the system’s non-linear nature.

![Orientation_KF](https://github.com/user-attachments/assets/84bf3ba7-ce35-4694-b76a-36267f84b493)

![Orientation_PF](https://github.com/user-attachments/assets/34fde6e3-586e-495f-a03f-3951dc5c5969)

#### 🔹 Phase 7: Orientation Bias

To reduce ambiguity, an orientation bias was added to the state.

This significantly improved convergence, especially for the EKF, leading to a sharp reduction in angular error.

![Orientation_KF_bias](https://github.com/user-attachments/assets/03726187-6c9f-4155-a081-1019490053a5)

![Orientation_PF_bias](https://github.com/user-attachments/assets/0d1f24f4-b5a8-47cf-ab06-3b9030609fb3)

#### 🔹 Phase 8: Compass and Full Observability

Finally, a compass (magnetometer) was introduced to provide direct orientation measurements.

With this addition, the system became fully observable in terms of orientation. The EKF achieved highly accurate estimates with a drastic reduction in error. The Particle Filter, however, showed larger errors due to noise in the particle distribution and difficulty in bias estimation.

![Orientation_KF_Obs_theta](https://github.com/user-attachments/assets/bca08f96-57a7-4ae7-bc35-65eeff74d3f4)

![Orientation_PF_Obs_theta](https://github.com/user-attachments/assets/3a4e98b8-5367-4dba-9035-6e9a73cd4470)

### ⚠️ GNSS Blackout Scenario

One of the most insightful experiments was simulating GNSS signal loss.

The EKF maintained a trajectory close to the ground truth, showing that it had effectively learned the system dynamics and bias terms. The Particle Filter, on the other hand, exhibited larger divergence due to the stochastic spread of particles.

However, once the sensor returned, the PF demonstrated its strength: rapid recovery. The resampling process assigns high weights to particles close to reality, pulling the estimate back abruptly.

![Blackout_KF](https://github.com/user-attachments/assets/a3f1bf03-3037-492e-8c34-86f01c955c85)

![Blackout_PF](https://github.com/user-attachments/assets/e93590f2-05e2-42e2-9a99-66b23e9851a1)

### 🔬 Key Challenges

Throughout the project, several challenges stood out:

* Tuning noise parameters (Q and R for Kalman, spread parameters for PF) was one of the most difficult tasks

* Handling asynchronous sensors required careful time management

* Understanding observability was essential to diagnose model limitations

Many of the hardest problems were not implementation-related, but conceptual.

### 🧠 Key Takeaways

* Small bias errors accumulate rapidly over time

* Filter performance is highly dependent on measurement quality

* EKF is efficient and stable but relies on good approximations

* Particle Filter is flexible but sensitive and computationally expensive

* System modeling is just as important as the estimation algorithm

### 🔮 Future Work

* Extend the model to 6 Degrees of Freedom (6-DoF)

* Test with real-world datasets (e.g., drones or autonomous vehicles)

* Improve Particle Filter robustness (resampling strategies, degeneracy reduction)

### 📌 Conclusion

This project represents a practical and incremental exploration of state estimation applied to inertial navigation.

More than implementing algorithms, the focus was on understanding how errors emerge, propagate, and can be mitigated. It highlights that there is no universal solution — each method comes with trade-offs, and performance depends heavily on the problem, sensors, and assumptions involved.
