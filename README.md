# Autonomous Turret with Face Detection for Localised Security

A two-axis pan-tilt turret that detects a human face in a live camera stream, aligns itself with the face using closed-loop control, and switches on a low-power laser pointer to indicate a simulated, non-lethal engagement.

This repository holds the source code for the final-year project *Development of an Autonomous Turret with Face Detection for Localised Security*, submitted in July 2025 to the Department of Mechatronics Engineering, Air Force Institute of Technology (AFIT), Kaduna, Nigeria, in partial fulfilment of the Bachelor of Engineering degree.

| | |
|---|---|
| Authors | Fleming Olabisi Airunugba, Jerome Idoko Onoja |
| Contributor | David Prince Hope (programming support: PID tuning and control logic) |
| Supervisor | Dr. Ameer Mohammed |
| Institution | Department of Mechatronics Engineering, Faculty of Air Engineering, AFIT Kaduna |
| Status | Completed prototype, July 2025 |

## Abstract

Conventional surveillance and turret systems depend on continuous human control, which limits response time. This project develops a low-cost autonomous turret that performs real-time face detection and simulated targeting with computer vision and embedded control. An ESP32-CAM module streams live video over Wi-Fi to a host computer, where the MediaPipe framework detects faces. The host computes the offset between the detected face and the centre of the frame, and a proportional-integral-derivative (PID) controller converts that offset into pan and tilt angles. The angles are sent over a UART serial link to an ESP32 microcontroller that drives two servo motors. When the face stays inside a tolerance zone around the frame centre, a laser module is switched on.

Unlike designs that run detection on the microcontroller or send control commands wirelessly, this design moves image processing to the host computer and uses a wired serial link for control. In testing, the system reached an average face-detection accuracy of 86% across 30 trials and a mean end-to-end latency of about 348 ms.

## System overview

```
ESP32-CAM  --(MJPEG over Wi-Fi/HTTP)-->  Host PC  --(UART serial, 115200 baud)-->  ESP32 DevKit
 camera                                  MediaPipe face detection                   2 servo motors (pan, tilt)
                                         PID control                                laser module
```

| Subsystem | Implementation | Source |
|---|---|---|
| Image acquisition | AI-Thinker ESP32-CAM serving an MJPEG stream | `firmware/esp32_cam_stream/` |
| Detection and control | Python: OpenCV, MediaPipe short-range face detector, PID controller | `host/face_tracking_turret.py` |
| Actuation | ESP32 DevKit V1 driving two MG996R servos and a KY-008 laser | `firmware/servo_controller/` |

Design detail is in [docs/system-design.md](docs/system-design.md). Test method and results are in [docs/results.md](docs/results.md).

## Hardware

| Component | Quantity | Function |
|---|---|---|
| ESP32-CAM (AI-Thinker) | 1 | Video capture and streaming |
| ESP32 DevKit V1 | 1 | Servo and laser control |
| MG996R servo motor | 2 | Pan and tilt axes |
| KY-008 laser module | 1 | Engagement indicator |
| External 5 V supply | 1 | Servo power (the servos must not be powered from the ESP32) |
| Perspex frame and conduit box | 1 | Pan-tilt structure and base |

Pin assignment on the ESP32 DevKit:

| Signal | GPIO |
|---|---|
| Pan servo | 23 |
| Tilt servo | 27 |
| Laser | 32 |

## Repository structure

```
firmware/
  esp32_cam_stream/     ESP32-CAM video streaming firmware (Arduino)
  servo_controller/     ESP32 servo and laser firmware (Arduino)
host/
  face_tracking_turret.py   Face detection, PID control and serial output
docs/
  system-design.md      Architecture, control method and serial protocol
  results.md            Test method, results and limitations
requirements.txt        Python dependencies
CITATION.cff            Citation metadata
```

## Setup

### 1. Camera firmware

1. Open `firmware/esp32_cam_stream/esp32_cam_stream.ino` in the Arduino IDE with the ESP32 board package installed.
2. Set `ssid` and `password` to your network. Set `local_IP` and `gateway` to match it.
3. Select the board **AI Thinker ESP32-CAM** and upload.
4. Confirm the stream opens at `http://<camera-ip>:81/stream`.

### 2. Servo controller firmware

1. Install the `ESP32Servo` library.
2. Open `firmware/servo_controller/servo_controller.ino`, select **ESP32 Dev Module** and upload.
3. Keep the board connected to the host computer by USB. This cable is the control link.

### 3. Host program

Python 3.9 to 3.11 is recommended for MediaPipe.

```bash
pip install -r requirements.txt
python host/face_tracking_turret.py
```

Before running, edit two values at the bottom of `host/face_tracking_turret.py`:

- `port` in `ESP32RobotController(...)`: the serial port of the ESP32 DevKit (for example `COM13` on Windows or `/dev/ttyUSB0` on Linux).
- `stream_url` in `FaceTrackingSystem(...)`, and `self.URL` in that class: the address of the ESP32-CAM.

Keyboard controls in the video window: `r` set resolution, `q` set JPEG quality, `a` toggle white balance, `x` or `Esc` quit.

## Results summary

| Test | Condition | Result |
|---|---|---|
| Face detection (30 trials, 2 subjects) | Well lit | 100% |
| | Low light | 80% |
| | Partial occlusion | 40 to 60% |
| | Average | 86% |
| Laser targeting (30 trials, 15 px tolerance) | Well lit | 100% |
| | Low light | 80% |
| | Head tilted 10 to 30 degrees | 40 to 60% |
| Servo response | Settling time | 300 to 360 ms |
| | Overshoot | below 5 degrees |
| End-to-end latency | Camera capture to servo movement | about 348 ms |

## Limitations

- Detection is reliable only at short range (about 30 to 80 cm) and degrades in low light and with partly covered faces.
- Image processing runs on a host computer; the turret is not self-contained.
- Mechanical backlash in the servos causes a steady-state error of 1 to 2 degrees.
- The system detects faces. It does not recognise or identify individuals.

## Future work

- Move detection and control onto a single board (ESP32-S3 or Raspberry Pi).
- Add object tracking (KCF or MOSSE) to follow moving targets between detections.
- Improve low-light performance with infrared illumination or adaptive lighting.
- Replace the servo power stage with regulated, current-limited drivers.

## Safety and ethics

This is an academic prototype. The laser is a low-power pointer used only to show alignment, and it must not be directed at eyes. The system has no projectile or harmful mechanism and is not intended for deployment against people.

## Citation

Airunugba, F. O., and Onoja, J. I. (2025). *Development of an Autonomous Turret with Face Detection for Localised Security*. B.Eng. project report, Department of Mechatronics Engineering, Air Force Institute of Technology, Kaduna, Nigeria.

## Acknowledgements

The project was supervised by Dr. Ameer Mohammed. David Prince Hope assisted with programming, in particular the PID tuning and parts of the control logic. The camera streaming firmware is based on the ESP32 CameraWebServer example by Espressif Systems; `app_httpd.cpp` is licensed by Espressif under the Apache License 2.0. Face detection uses Google MediaPipe.
