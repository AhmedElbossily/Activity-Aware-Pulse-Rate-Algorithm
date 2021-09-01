# Pulse Rate Algorithm Overview
This project creates a pulse rate estimation algorithm for a wrist-wearable device.

## Introduction
A core feature that many users expect from their wearable devices is pulse rate estimation. Continuous pulse rate estimation can be informative for many aspects of a wearer's health. Pulse rate during exercise can be a measure of workout intensity and resting heart rate is sometimes used as an overall measure of cardiovascular fitness. 


## Physiological Mechanics of Pulse Rate Estimation
Pulse rate is typically estimated by using the **PPG sensor**. When the ventricles contract, the capilaries in the wrist fill with blood. The (typically green) light emitted by the PPG sensor is absorbed by red blood cells in these capilaries and the photodetector will see the drop in reflected light. When the blood returns to the heart, fewer red blood cells in the wrist absorb the light and the photodetector sees an increase in reflected light. The period of this oscillating waveform is the pulse rate.
![](https://github.com/AhmedElbossily/Activity-Aware-Pulse-Rate-Algorithm/blob/main/ppg_mechanics.png?raw=true)

However, the heart beating is not the only phenomenon that modulates the PPG signal. Blood in the wrist is fluid, and arm movement will cause the blood to move correspondingly. During exercise, like walking or running, we see another periodic signal in the PPG due to this arm motion. Our pulse rate estimator has to be careful not to confuse this periodic signal with the pulse rate.

We can use the accelerometer signal of our wearable device to help us keep track of which periodic signal is caused by motion. Because the accelerometer is only sensing arm motion, any periodic signal in the accelerometer is likely not due to the heart beating, and only due to the arm motion. If our pulse rate estimator is picking a frequency that's strong in the accelerometer, it may be making a mistake.

All estimators will have some amount of error. How much error is tolerable depends on the application. If we were using these pulse rate estimates to compute long term trends over months, then we may be more robust to higher error variance. However, if we wanted to give information back to the user about a specific workout or night of sleep, we would require a much lower error.

## Algorithm Specifications:
  * estimates pulse rate from the PPG signal and a 3-axis accelerometer.
  * assumes pulse rate will be restricted between 40BPM (beats per minute) and 240BPM
  * produces an estimation confidence. A higher confidence value means that this estimate should be more accurate than an estimate with a lower confidence value.
  * produces an output at least every 2 seconds.  

## To run this code
- You need to import
 - glob
 - numpy
 - scipy
 - collections
 - matplotlib
- The notebook consists of two main code cells.
- The first cell contains all the algorithm functions.
- The second one contains the Evaluate method which kicks off the algorithm.
-
## The dataset Troika[1] was used to build the algorithm.
- The dataset contains (ECG signal, PPG signal, (x,y,z) accelerattion signal)
- he ground truth of the hear beat was obtained from the ECG signal
- The magnitude acceleration was obtained from (x,y,z) acceleration signals
- All signals are filtered by a band bass filter 40:240 BPM
- The dataset consists of 24 files.
- Files with similar name "DATA_01_TYPE01" contain a variable 'sig'. It ehas 6 rows. The first row is a simultaneous recording of ECG. The second row and the third row are two channels of PPG. The last three rows are simultaneous recordings of acceleration data (in x-, y-, and z-axis).
- Files with a similar name "REF_01_TYPE01" contain a variable 'BPM0', which holds the ground-truth heart rate.
- All signals were sampled at 125 Hz.
- **Short-comings of the dataset**: The length of some "REF_01_TYPE01" files are not equal to the length of total 8 seconds frames in the "DATA_01_TYPE01" files.

## Algorithhm Description

### How the algorithm works?
- The algorithm loads the database into two NumPy arrays. One for data and another for references.
- For every file in the database, the data is divided into time frames. Every frame consists of 8 seconds.
- These frames are stored in a NumPy array called features.
- The true heartbeat rate for every frame is stored in a NumPy array called labels.
- All feature signals are filtered by a bandpass filter (40:240) BPM.
- The algorithm starts by converting the frame signal from the time domain to the frequency domain.
- For every time window, FFT is used to calculate the spectrum of each channel of acceleration data and PPG signal, from which dominant frequencies are determined.
- The dominant frequencies of the acceleration are the ones corresponding to the spectral peaks with an amplitude larger than 50% of the maximum amplitude in a given spectrum.
- The dominant frequencies of the acceleration are removed from the PPG signal.
- The dominant frequencies of the acceleration may have values that equal to the PPG dominant frequency. To prevent removing the dominant frequency of the PPG signal, the dominant frequency of the previous frame was used to remove any dominant acceleration frequencies that equal it (+- 12 BPM).
- The heartbeat rate was obtained from the dominant frequency of the PPG signal
- The algorithm contains a method for heartbeat rate verification. This method prevents a large change in the estimated BPM values in two successive time windows.
- The change of BPM values in two successive time windows was prevented to exceed 10 BPM.

### The specific aspects of the physiology of heartbeat rate
The heartbeat rate is always between 40:240 BPM, so a bandpass filter was used to remove frequencies out of this range.
The change of BPM values in two successive time windows rarely exceeds 10 BPM, so a heartbeat rate verification method was designed based on this concept.

### Confidence rate
The energy concentration in the frequency spectrum around the estimated pulse rate is used to obtain a confidence rate for every estimate. This was done by summing frequency spectrum near the pulse rate estimate and dividing it by the sum of the entire spectrum. A high confidence rate means better estimations because it means that the signal has low noise, and there is one main dominant frequency in the signal.

### Algorithm outputs
The mean absolute error of the best 90% of heartbeat rate estimates for the entire dataset.

### caveats on algorithm outputs
The algorithm generates output every 2 seconds.

### common failure modes
The common failure takes place when the heartbeat of the first time frame was set wrong. This could happen if there is a strong acceleration signal in the first time frame. To avoid that, the subject is required to hold his hand for the first 8 seconds.
When there is no contact between the PPG sensor and the subject's skin.

### Algorithm Performance
The mean absolute error (MAE) of the best 90 estimates was used to calculate the algorithm performance, and it was 6.7 through the entire dataset.
The algorithm was tested on another dataset, and the (MAE) of the best 90 estimates was 6.83
The energy concentration in the frequency spectrum around the estimated pulse rate is used to obtain a confidence rate for every estimate. This was done by summing frequency spectrum near the pulse rate estimate and dividing it by the sum of the entire spectrum.

### Citations
Zhilin Zhang, Zhouyue Pi, Benyuan Liu, ‘‘TROIKA: A General Framework for Heart Rate Monitoring Using Wrist-Type Photoplethysmographic Signals During Intensive Physical Exercise,’’IEEE Trans. on Biomedical Engineering, vol. 62, no. 2, pp. 522-531, February 2015. Link

1. **Troika** - Zhilin Zhang, Zhouyue Pi, Benyuan Liu, ‘‘TROIKA: A General Framework for Heart Rate Monitoring Using Wrist-Type Photoplethysmographic Signals During Intensive Physical Exercise,’’IEEE Trans. on Biomedical Engineering, vol. 62, no. 2, pp. 522-531, February 2015. Link
