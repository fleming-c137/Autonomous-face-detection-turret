# System design

## 1. Architecture

The system separates perception, control and actuation across three devices.

1. **ESP32-CAM** captures video and serves it as an MJPEG stream over HTTP on port 81.
2. **Host computer** reads the stream, detects faces, computes the control output and sends angle commands.
3. **ESP32 DevKit** receives angle commands over UART and drives the pan servo, the tilt servo and the laser.

Running detection on the host removes the processing limit of the microcontroller. Using a wired serial link for control, in place of the WebSocket link used in an earlier iteration, removed the variable delay observed over Wi-Fi.

## 2. Face detection

Each frame is mirrored horizontally, converted to RGB and passed to the MediaPipe short-range face detector (`model_selection=0`, `min_detection_confidence=0.5`). When several faces are present, the face with the largest bounding box is selected as the target. The target point is the centre of that bounding box.

## 3. Control

The error on each axis is the distance between the frame centre and the target point, expressed as a percentage of the frame width (pan) or height (tilt):

```
e_x = 100 * (x_centre - x_face) / frame_width
e_y = 100 * (y_centre - y_face) / frame_height
```

Each axis has an independent PID controller:

```
u(t) = Kp * e(t) + Ki * integral(e dt) + Kd * de/dt
```

The integral term is clamped to prevent wind-up, and the output is limited to plus or minus 90 degrees. The commanded angle is `90 - u`, limited to 0 to 180 degrees for pan and 0 to 170 degrees for tilt.

**Dead zone.** If the absolute error on an axis is below 8% of the frame, that axis is not updated. This prevents oscillation around the target.

**Smoothing.** Angle commands are smoothed twice: on the host (exponential moving average, alpha 0.3, at most 5 degrees change per update, one update every 0.2 s) and on the microcontroller (exponential moving average, alpha 0.2, at most 3 degrees per update, at least 100 ms between updates).

**Controller gains.** The host program sets Kp 0.001, Ki 0.1 and Kd 2 on both axes. Gains depend on the mechanical build and should be re-tuned for any new assembly.

## 4. Serial protocol

The link runs at 115200 baud. Each command is one ASCII line ending in a newline character.

| Command | Meaning | Valid range |
|---|---|---|
| `pan:X` | Move the pan axis towards X degrees | 0 to 180 |
| `tilt:Y` | Move the tilt axis towards Y degrees | 0 to 170 |
| `deadzone` | Target is centred on both axes; hold position | none |

The microcontroller replies `ACK` to a valid command and `ERROR` to an invalid one.

## 5. Engagement logic

The laser is switched on after three consecutive `deadzone` commands, which means the target has stayed centred on both axes. Any later `pan` or `tilt` command switches the laser off and resets the count.

## 6. Fault handling

The host program reconnects to the serial port and to the video stream with exponential back-off (starting at 3 s, up to 30 s, five attempts). It records the frame rate to `fps_log.txt` during operation.

## 7. Mechanical structure

The pan-tilt frame was designed in SolidWorks and built from Perspex, mounted on an electrical conduit box that houses the controller and wiring. The servos are powered from an external 5 V supply because their current draw causes brown-outs when they share the microcontroller supply.
