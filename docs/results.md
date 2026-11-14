# Test method and results

All figures are taken from Chapter 4 of the project report. Tests were carried out indoors with two human subjects.

## 1. Face detection accuracy

**Method.** Fifteen detection attempts per subject (30 in total): five in good light, five in low light and five with the face partly covered by a hand, glasses or a mask. Frames were streamed at 320 x 240. A detection was counted as correct when MediaPipe returned a bounding box on a face that was present. Bounding-box error is the pixel distance between the detected and the manually marked face centre.

| Condition | Trials | True positive | False positive | False negative | Precision | Recall | F1 | Accuracy | Mean box error (px) |
|---|---|---|---|---|---|---|---|---|---|
| Well lit, subject 1 | 5 | 5 | 0 | 0 | 1.00 | 1.00 | 1.00 | 100% | 3.2 |
| Low light, subject 1 | 5 | 4 | 0 | 1 | 1.00 | 0.80 | 0.89 | 80% | 4.8 |
| Occlusion, subject 1 | 5 | 3 | 1 | 2 | 0.75 | 0.60 | 0.67 | 60% | 6.1 |
| Well lit, subject 2 | 5 | 5 | 0 | 0 | 1.00 | 1.00 | 1.00 | 100% | 2.9 |
| Low light, subject 2 | 5 | 4 | 0 | 1 | 1.00 | 0.80 | 0.89 | 80% | 4.5 |
| Occlusion, subject 2 | 5 | 2 | 1 | 3 | 0.67 | 0.40 | 0.50 | 40% | 6.7 |

The report gives the average detection accuracy as 86%.

## 2. Targeting accuracy

**Method.** Each subject stood about 1.5 m from the camera. A trial was counted as a hit when the laser dot landed within 15 pixels of a reference marker at the centre of the face.

| Condition | Trials | Hits | Accuracy | Mean deviation (px) |
|---|---|---|---|---|
| Well lit, subject 2 | 5 | 5 | 100% | 4.3 |
| Low light, subject 2 | 5 | 4 | 80% | 6.9 |
| Head tilt 10 to 30 degrees, subject 2 | 5 | 3 | 60% | 9.2 |
| Well lit, subject 1 | 5 | 5 | 100% | 4.0 |
| Low light, subject 1 | 5 | 4 | 80% | 6.7 |
| Head tilt 10 to 30 degrees, subject 1 | 5 | 2 | 40% | 10.5 |

## 3. Servo response

**Method.** Each axis was commanded to fixed angles five times. Settling was defined as staying within 2 degrees of the target.

| Axis | Target (degrees) | Overshoot (degrees) | Steady-state error (degrees) |
|---|---|---|---|
| Pan | 90 | 3.5 | 1.2 |
| Pan | 150 | 4.1 | 1.5 |
| Tilt | 90 | 3.2 | 1.0 |
| Tilt | 150 | 4.5 | 1.7 |

The servos began moving within 200 to 220 ms and settled within 300 to 360 ms.

## 4. System latency

**Method.** The system was filmed at 120 frames per second while a face entered the camera view. Delays were measured frame by frame.

| Stage | Mean delay (ms) |
|---|---|
| ESP32-CAM capture | 90 |
| Frame transmission over Wi-Fi | 25 |
| MediaPipe detection (Intel Core i5) | 42 |
| PID computation | 3 |
| Command dispatch over UART | 8 |
| Servo response | 180 |
| **Total** | **348** |

## 5. Limitations of the evaluation

- The sample is small: 30 trials per test and two subjects.
- All tests were indoors at short range with a stationary or slowly moving subject.
- Ground-truth positions were marked by hand.
- The results describe the specific prototype that was built; servo backlash and frame alignment vary between assemblies.
