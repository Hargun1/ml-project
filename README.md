# UAV Fault Detection and Live Drone Visualization

This project demonstrates fault detection and diagnosis for an unmanned aerial vehicle using machine learning and deep learning. It combines simulated UAV sensor data, trained accelerometer and gyroscope fault classifiers, a Flask web dashboard, live signal graphs, and a graph-driven drone visual that makes the detected fault behavior easier to explain during a presentation.

## Project Goal

UAVs depend heavily on sensor feedback for stable flight. If the accelerometer or gyroscope starts producing faulty values, the controller can make unsafe decisions. This project detects those sensor faults and shows the result in a visual dashboard.

The demonstration answers three questions:

1. What is the current accelerometer behavior?
2. What is the current gyroscope behavior?
3. How would the UAV visually react to that fault during flight?

## Main Features

- Detects UAV sensor faults from accelerometer and gyroscope readings.
- Uses a CNN-LSTM deep learning model for accelerometer classification.
- Uses a machine learning model for gyroscope classification.
- Streams live AccX and GyrX values into browser graphs.
- Shows prediction labels for both sensor streams.
- Includes demo scenario buttons for presentation control.
- Adds a live drone canvas visual that reacts to graph data and selected fault mode.
- Displays controller actions such as filtering, bias trim, hold hover, backup estimation, and emergency landing.

## Repository Structure

```text
.
├── README.md
├── .gitignore
└── Fault-Detection-in-UAVs-using-Deep-Learning-and-Machine-Learning/
    ├── README.md
    ├── ML_DL/
    │   ├── README.md
    │   ├── Acc_Models/
    │   ├── Acc_Test_Scripts/
    │   ├── Acc_Training_Scripts/
    │   ├── Gyr_Models/
    │   ├── Gyr_Test_Scripts/
    │   └── Gyr_Training_Scripts/
    ├── parrotMinidroneWaypointFollower/
    │   ├── README.md
    │   ├── dataCollection/
    │   ├── controller/
    │   ├── mainModels/
    │   ├── nonlinearAirframe/
    │   └── support/
    └── Demonstration/
        ├── app.py
        ├── config.yaml
        ├── requirements.txt
        ├── templates/
        │   └── index.html
        ├── Acc_CNN_LSTM_5_Class_64_28-07-2021.h5
        ├── gyrModel.joblib
        ├── Acc_Data.csv
        ├── Gyr_Data.csv
        └── Gyr_test.csv
```

## How The Project Works

The project has three major stages.

### 1. Data Collection

The `parrotMinidroneWaypointFollower` folder contains a MATLAB/Simulink-based UAV simulation. It is used to generate normal and faulty UAV flight data.

The simulation records sensor and flight-control values such as:

- `AccX`, `AccY`, `AccZ`
- `GyrX`, `GyrY`, `GyrZ`
- `Altitude`
- motor command values
- `Roll`, `Pitch`, `Yaw`
- fault label

Faults are injected into the accelerometer or gyroscope sensor path. The collected data is then converted into machine-learning-ready datasets.

### 2. Model Training

The `ML_DL` folder contains notebooks and model artifacts for training and testing.

The project separates accelerometer and gyroscope fault detection:

- Accelerometer faults are classified using a CNN-LSTM sequence model.
- Gyroscope faults are classified using a machine learning model saved as `gyrModel.joblib`.

The supported output classes are:

| Label | Fault Type |
|---|---|
| 0 | Normal |
| 1 | Noise |
| 2 | Stuck-At |
| 3 | Offset |
| 4 | Packet Drop |

### 3. Live Demonstration

The `Demonstration` folder contains the Flask dashboard.

When the web app runs:

1. `app.py` loads `config.yaml`.
2. It reads accelerometer and gyroscope CSV files.
3. It preprocesses the sensor windows.
4. It loads the trained accelerometer and gyroscope models.
5. The `/data` endpoint returns live graph data and model predictions.
6. `templates/index.html` updates the graphs, prediction labels, scenario panel, and drone visual.

## Demonstration App Flow

The Flask app has two important routes:

| Route | Purpose |
|---|---|
| `/` | Loads the dashboard page and initializes data. |
| `/data` | Streams the next data window, predictions, and graph values as JSON. |

The `/data` response follows this structure:

```text
[
  time_points,
  accelerometer_prediction,
  gyroscope_prediction,
  AccX_values,
  GyrX_values
]
```

The browser uses this response to update:

- AccX graph
- GyrX graph
- accelerometer prediction text
- gyroscope prediction text
- drone movement
- drone status and response messages

## Drone Visual Behavior

The drone visual was added to make the live graph output easier to demonstrate.

It is not a physics simulator. It is a presentation visual that maps sensor graph behavior and fault categories into clear flight behavior.

| Fault Mode | Drone Behavior | Demonstration Meaning |
|---|---|---|
| Normal | Smooth motion and stable altitude | UAV is following the trajectory correctly. |
| Noise | Drone shakes and oscillates | Sensor noise is disturbing the controller input. |
| Offset | Drone drifts away from the path | Sensor bias is creating incorrect state estimation. |
| Stuck-At | Drone movement becomes limited or locked | Sensor readings are stale and cannot represent real motion. |
| Packet Drop | Drone blinks/steps between updates | Sensor packets are missing intermittently. |
| Crash Risk | Drone drops and shows critical warning | Emergency recovery or controlled landing is needed. |

The visual uses:

- AccX graph values to influence vertical motion and altitude.
- GyrX graph values to influence horizontal drift and roll.
- Prediction labels to decide the severity state.
- Scenario buttons to force a presentation mode when live model output is not enough for explanation.

## Demo Scenario Buttons

The dashboard includes these modes:

- Live Model
- Normal Flight
- Noise Fault
- Offset Fault
- Stuck-At Fault
- Packet Drop Fault
- Crash Risk

`Live Model` uses the actual prediction result from the trained models. The other buttons override the displayed behavior so the presenter can explain each fault clearly.

## Configuration

The dashboard is configured through:

```text
Fault-Detection-in-UAVs-using-Deep-Learning-and-Machine-Learning/Demonstration/config.yaml
```

Important values:

```yaml
Data_acc:
  data_file: "Acc_Data.csv"

Data_gyr:
  data_file: "Gyr_Data.csv"

model:
  model_acc: "Acc_CNN_LSTM_5_Class_64_28-07-2021.h5"
  model_gyr: "gyrModel.joblib"
```

The config also defines selected columns, output labels, time-window length, and streaming step size.

## How To Run The Demonstration

Open a terminal in:

```text
Fault-Detection-in-UAVs-using-Deep-Learning-and-Machine-Learning/Demonstration
```

Create or activate a Python environment, then install dependencies:

```bash
pip install -r requirements.txt
```

Run the Flask app:

```bash
python -m flask --app app run --host 127.0.0.1 --port 5000
```

Open:

```text
http://127.0.0.1:5000/
```

## Required Runtime Files

The demonstration expects these files inside the `Demonstration` folder:

```text
Acc_Data.csv
Gyr_Data.csv
Gyr_test.csv
Acc_CNN_LSTM_5_Class_64_28-07-2021.h5
gyrModel.joblib
```

If any required file is missing, the dashboard shows an error message instead of silently failing.

## Technology Stack

- Python
- Flask
- Pandas
- NumPy
- SciPy
- scikit-learn
- TensorFlow/Keras
- Joblib
- Highcharts
- HTML/CSS/JavaScript Canvas
- MATLAB/Simulink for UAV data generation

## Notes

- Large CSV files are ignored by `.gitignore` to avoid unnecessary repository size.
- Local runtime logs under `.logs/` are ignored.
- The drone visual is designed for explanation and demonstration, not for aerodynamic validation.
- TensorFlow startup can take some time when the Flask server starts.

