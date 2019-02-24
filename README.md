# Robotics-Sensor-Fusion-02-EKF-Lidar-and-Radar-Fusion

Udacity Self-Driving Car Engineer Nanodegree:  Lidar and Radar Fusion with Kalman Filters in C++.

The task is to track a prdestrain moving in front of our autonomous vehicle.

This project can use multiple data sources originating from different sensors to estimate a more accurate object state.

## Overview
The Kalman Filter algorithm will go through the following steps:
- First measurement.
- initialize state and covariance matrices.
- Predict.
- Update.
- Do another predict and update step (when receive another sensor measurement).

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Overview%20of%20the%20Kalman%20Filter%20Algorithm%20Map.png" width = "70%" height = "70%" div align=center />

## Two-step estimation problem

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Estimation%20Problem%20Refresh.png" width = "60%" height = "60%" div align=center />

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Estimation%20Problem%20Refresh2.png" width = "60%" height = "60%" div align=center />

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Estimation%20Problem%20Refresh3.png" width = "60%" height = "60%" div align=center />

## State Prediction

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/State%20Prediction.png" width = "60%" height = "60%" div align=center />

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/State%20Prediction2.png" width = "60%" height = "60%" div align=center />

Because our state vector only tracks position and velocity, we are modeling acceleration as a random noise. 

```cpp
void KalmanFilter::Predict() {
	x_ = F_ * x_;
	MatrixXd Ft = F_.transpose();
	P_ = F_ * P_ * Ft + Q_;
```

## Lidar Measurements

- ``z``measurement vector
- ``x``state vector
- ``R``measurement covariance matrix
- ``H``measurement matrix: Find the right H matrix to project from a 4D state to a 2D observation space.

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Laser%20Measurements.png" width = "50%" height = "50%" div align=center />

### Process Covariance Matrix

Because our state vector only tracks position and velocity, we are **modeling acceleration as a random noise**. 

<img src="https://github.com/ChenBohan/Robotics-Sensor-Fusion-02-EKF-Lidar-and-Radar-Fusion/blob/master/readme_img/Process%20Covariance%20Matrix" width = "60%" height = "60%" div align=center />

<img src="https://github.com/ChenBohan/Robotics-Sensor-Fusion-02-EKF-Lidar-and-Radar-Fusion/blob/master/readme_img/Process%20Covariance%20Matrix2.png" width = "60%" height = "60%" div align=center />

<img src="https://github.com/ChenBohan/Robotics-Sensor-Fusion-02-EKF-Lidar-and-Radar-Fusion/blob/master/readme_img/Process%20Covariance%20Matrix3.png" width = "60%" height = "60%" div align=center />

<img src="https://github.com/ChenBohan/Robotics-Sensor-Fusion-02-EKF-Lidar-and-Radar-Fusion/blob/master/readme_img/Process%20Covariance%20Matrix4.png" width = "60%" height = "60%" div align=center />

[Details: Udacity Process Covariance Matrix](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/3612b91d-9c33-47ad-8067-a572a6c93837/concepts/1ac6e0ac-1809-4864-b58f-870d6bda9b25)

### Code
```cpp
  // compute the time elapsed between the current and previous measurements
  // dt - expressed in seconds
  float dt = (measurement_pack.timestamp_ - previous_timestamp_) / 1000000.0;
  previous_timestamp_ = measurement_pack.timestamp_;

  float dt_2 = dt * dt;
	float dt_3 = dt_2 * dt;
	float dt_4 = dt_3 * dt;

  // 1. Modify the F matrix so that the time is integrated
  kf_.F_(0, 2) = dt;
	kf_.F_(1, 3) = dt;

  // 2. Set the process covariance matrix Q
  kf_.Q_ = MatrixXd(4, 4);
	kf_.Q_ << dt_4 / 4 * noise_ax, 0, dt_3 / 2 * noise_ax, 0,
	          0, dt_4 / 4 * noise_ay, 0, dt_3 / 2 * noise_ay,
	          dt_3 / 2 * noise_ax, 0, dt_2*noise_ax, 0,
	          0, dt_3 / 2 * noise_ay, 0, dt_2*noise_ay;

  // 3. Call the Kalman Filter predict() function
  kf_.Predict();

  // 4. Call the Kalman Filter update() function
  //      with the most recent raw measurements_
  kf_.Update(measurement_pack.raw_measurements_);
```

### Disadvantages

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Disadvantages.png" width = "50%" height = "50%" div align=center />

It works quite well when the pedestrian is moving along the straght line.

However, our linear motion model is not perfect, especially for the scenarios when the pedestrian is moving along a circular path.

To solve this problem, we can predict the state by using a more complex motion model such as the circular motion.

## Radar Measurements

The radar can directly measure the object ``range``, ``bearing``, ``radial velocity``.

We use the extended kalman filter in Radar Measurements for non-linear function.

What we change is we simply use non-linear function f(x) to predict the state, and h(X) to compute the measurement error.

So we first linearize the non-linear prediction and measurement functions, and then use the same mechanism to estimate the new state.

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Generalization.png" width = "50%" height = "50%" div align=center />

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Radar%20Measurements.png" width = "50%" height = "50%" div align=center />

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Radar%20Measurements2.png" width = "50%" height = "50%" div align=center />

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Radar%20Measurements3.png" width = "30%" height = "30%" div align=center />

Extended Kalman filter (EKF) is the nonlinear version of the Kalman filter which linearizes about an estimate of the current mean and covariance.

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Extended%20Kalman%20Filter.png" width = "50%" height = "50%" div align=center />

Calculate the Jacobian matrix

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Jacobian%20Matrix.png" width = "50%" height = "50%" div align=center />

```cpp
//pre-compute a set of terms to avoid repeated calculation
float c1 = px*px+py*py;
float c2 = sqrt(c1);
float c3 = (c1*c2);

//check division by zero
if(fabs(c1) < 0.0001){
	cout << "CalculateJacobian () - Error - Division by Zero" << endl;
	return Hj;
}

//compute the Jacobian matrix
Hj << (px/c2), (py/c2), 0, 0,
	-(py/c1), (px/c1), 0, 0,
	py*(vx*py - vy*px)/c3, px*(px*vy - py*vx)/c3, px/c2, py/c2;
```

## Evaluating KF Performance

<img src="https://github.com/ChenBohan/Auto-Car-Sensor-Fusion-02-Lidar-and-Radar-Fusion/blob/master/readme_img/Evaluating%20KF%20Performance.png" width = "50%" height = "50%" div align=center />

```cpp
VectorXd CalculateRMSE(const vector<VectorXd> &estimations,
	const vector<VectorXd> &ground_truth) {

	VectorXd rmse(4);
	rmse << 0, 0, 0, 0;

	// check the validity of the following inputs:
	//  * the estimation vector size should not be zero
	//  * the estimation vector size should equal ground truth vector size
	if (estimations.size() != ground_truth.size()
		|| estimations.size() == 0) {
		cout << "Invalid estimation or ground_truth data" << endl;
		return rmse;
	}

	//accumulate squared residuals
	for (unsigned int i = 0; i < estimations.size(); ++i) {

		VectorXd residual = estimations[i] - ground_truth[i];

		//coefficient-wise multiplication
		residual = residual.array()*residual.array();
		rmse += residual;
	}

	//calculate the mean
	rmse = rmse / estimations.size();

	//calculate the squared root
	rmse = rmse.array().sqrt();

	//return the result
	return rmse;
}
```
