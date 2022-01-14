## Can physiological performance metrics like oxygen consumption and heart rate be reliably predicted with neural networks?
---

### Original Study
The original inspiration for this project was a research study that I came across: "Predicting oxygen consumption in high intensity cycling exercise". The study took 7 test subjects and had them each complete 4 different tests (workouts) on a stationary bicycle in a lab environment.
<br>
- Incremental Resistance Test: Test subjects rode a stationary bicycle, and the pedaling resistance was periodically increased over a 30 minute period.
- Wingate Test: An all out 30 second sprint on a stationary bike.
- Interval Test 1: This workout had test subjects complete a few high exertion intervals in order to capture their physiological response to repeated increases and decreases in power output.
- Interval Test 2: This test was very similar to the previous one, but with a few minor changes. <br>

During each test, the following data was collected: Time, Power, Oxygen, Cadence, Heart Rate, and Respiratory Frequency. The goal was to develop a model that would predict the oxygen consumption of an athlete during a cycling workout. For each test subject, three tests were used to train a Recurrent Neural Network (RNN) model with Long Short Term Memory (LSTM) layers. The original model that was developed produced good results, with an $R^2$ score of just over 0.9 for some tests. The RNN model that was referenced in the original study used the following structure:

| Layer(Type) | Output Shape | N Parameters |
| ----------- | ------------ | ------------ |
| LSTM 1      | 10 x 70 x 32 | 4736         |
| LSTM 2      | 10 x 70 x 32 | 8320         |
| LSTM 3      | 10 x 32      | 8320         |
| Dense 1     | 10 x 10      | 330          |
| Dense 2     | 10 x 1       | 11           |

<br>
Instead of just reusing the model structure defined in the original study, a more complex model was developed in this project that is focused more on optimizing the accuracy of the model at the expense of increased complexity and computing time. It was able to acheive much higher $R^2$ scores, coming in at 0.997 on training data and 0.994 on testing data. This RNN structure is as follows:

| Layer (type)                 | Output Shape              | Param #   |
| ---------------------------- | ------------------------- | --------- |
| LSTM                 | (None, 70, 128)           | 69120     |
| LeakyReLU      | (None, 70, 128)           | 0         |
| LSTM                | (None, 70, 128)           | 131584    |
| LeakyReLU    | (None, 70, 128)           | 0         |
| Dropout           | (None, 70, 128)           | 0         |
| LSTM               | (None, 64)                | 49408     |
| Dropout       |   (None, 64)            |    0         |
| Dense (Output) |               (None, 1)     |            65      |  


<p style="text-align: center;">
Total params: 250,177 <br>
Trainable params: 250,177 <br>
Non-trainable params: 0 <br>
Epochs: 250 <br>
Batch Size: 32 <br>
</p>

---

### Golden Cheetah (GC) Study

It was interesting to see how the original study could be reproduced and even improved upon by modifying the RNN structure, but how well would a similar model work on real world data? Could it be used to reliably predict heart rate? <br>
<br>
##### Data Source
The data used in this portion of this project has been collected from an open source platform called 'Golden Cheetah' (GC). This platform allows users to upload data files from their workouts, and provides tools for users to analyze their data however they see fit. GC keeps records of all their users training sessions open to the public, and hosts the data on an AWS server for anyone who wants to access it. The users who upload workouts to GC are generally cyclists and triathletes, and the potential features in each data set are: time, distance, power, heart rate, cadence and altitude.

##### How is this useful?
During a race or even a training session, athletes may want to keep their heart rate under a certain level (eg. LTHR) in order to properly pace themselves and avoid a decline in performance. Heart rate typically lags behind the amount of physical exertion that caused it, so an increase in effort will not always result in an immediate increase in heart rate. By utilizing a power meter and cadence sensor, the RNN model in the GC section of this project can predict what an athletes future heart rate will be. This knowledge allows athletes to adjust their power output in order to maintain their target heart rate more consistently. Without this feature, it can be difficult to maintain a target heart rate because an adjustment in power output is typically not reflected in heart rate immediately.

##### Data preprocessing and cleaning
The data collected from GC had to be analyzed in order to create train and test data for the model that was only from cycling workouts on a stationary trainer, and included power, heart rate, and cadence data. The notebook 'gc_data_cleaning_preprocessing' handled this issue. It looked at a file filled with csv files of the workouts that one person completed and uploaded to GC. Each of those files was analyzed and only the useable files were kept for the model to use. Because there were such a large number of useable files (workouts) from this athlete, the selection was narrowed down to only useable workouts from June and July 2019. It was beneficial to be able to have a lot of data from such a narrow timeframe because the fitness level of the athlete was unlikely to have varied significantly during that time period. Using workouts that are spread out over the course of a year could yield unpredictable results due to changing fitness levels throughout the year. Once the desired number of useable workouts were selected, the files had to be cleaned and concatenated into training and testing datasets that were ready for modeling.

##### Modeling
The RNN model that was developed in this project for the GC data shares a nearly identical structure to that of the RNN used for the original study data. Each Epoch took significantly longer to run, but only 50 epochs were required in order to acheive $R^2$ score of 0.993 on both train and test data. Visualizations of this models performance can be found throughout the Jupyter Notebook under 'gc_study/code/gc_rnn_model.ipynb'.

---
### Potential for improvement
- The models currently predict one second (row) ahead, but the ability to reliably predict 3 or even 5 seconds ahead would be much more beneficial to users.
- The ability to deploy the GC heart rate prediction RNN model in a mobile phone app that would monitor your power, cadence, and heart rate data, in order to provide real time heart rate predictions would be very helpful to cyclists who are interested in heart rate bases pacing strategies, particularly if it could be deployed in a cycling computer.

---
### References

##### Original Study
-   https://www.researchgate.net/publication/339890475_Estimating_an_individual%27s_oxygen_uptake_during_cycling_exercise_with_a_recurrent_neural_network_trained_from_easy-to-obtain_inputs_A_pilot_study
- 
https://www.kaggle.com/andreazignoli/cycling-vo2/activity

##### GC Study
- http://www.goldencheetah.org/
- https://github.com/GoldenCheetah/OpenData
- Data retrieved from: http://goldencheetah-opendata.s3-website-us-east-1.amazonaws.com/?prefix=data/
    - file name: 'ffea5c2b-1898-4cbd-8fb4-64502aa4e962.zip'