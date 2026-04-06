# Sign-Based-Command-Recognition-and-Execution-for-Autonomous-Robot-Navigation

Team Members:
- Ty Carlisle, tycarlis@buffalo.edu

---

## Project Objective
The goal of this project is to develop a vision-based system that enables an autonomous robot car to read text on physical signs and execute corresponding commands. Building on in-class assignments involving camera-based object detection, this project will use optical character recognition (OCR) to detect and interpret a "STOP" sign, causing the robot to halt its movement upon recognition. This extends the existing coursework by shifting from pre-programmed visual cues (such as line tracking) to real-time text interpretation and decision-making.

## Contributions
While the class assignments cover foundational concepts like line tracking and object detection, none involve reading and interpreting text from the environment to drive robot behavior. This project bridges computer vision and autonomous decision-making by introducing OCR as an input method for robot control. The approach demonstrates a practical, real-world application — reading and obeying signage — that could be extended to more complex command sets in future work.

## Project Plan
- Build on existing class assignments for camera integration and basic motor control.
- Use **OpenCV** for image preprocessing (grayscale conversion, thresholding, region-of-interest detection) to isolate sign text from the camera feed.
- Use **Tesseract OCR** to perform character recognition on the detected sign region.
- Implement control logic that maps recognized text (e.g., "STOP") to a motor command (e.g., halt).
- Reference OpenCV and Tesseract documentation, as well as in-class lecture materials and assignments on camera-based detection.

## Milestones/Schedule Checklist
- [ ] See what is already available for sign detection
- [x] Complete this proposal document. *Due March 31*
- [ ] Review and document existing class assignment code for camera detection and motor control. *TC, April 4*
- [ ] Set up OpenCV pipeline for image preprocessing (grayscale, thresholding, ROI extraction). *TC, April 9*
- [ ] Integrate Tesseract OCR and test text recognition on static sign images. *TC, April 14*
- [ ] Combine OCR output with robot control logic (detect "STOP" → halt motors). *TC, April 18*
- [ ] Create progress report. *Due April 21*
- [ ] Test full pipeline on the physical robot with a printed STOP sign. *TC, April 25*
- [ ] Tune for reliability (distance, angle, lighting variations). *TC, May 1*
- [ ] Create final presentation. *Due May 5*
- [ ] Refine system based on presentation feedback. *TC, May 10*
- [ ] Provide system documentation (README.md). *Due May 15*

## Measures of Success
- [ ] OpenCV pipeline successfully isolates the sign region from the camera feed.
- [ ] Tesseract OCR correctly reads "STOP" from a captured image in a static test.
- [ ] Robot successfully detects and reads a "STOP" sign using the onboard camera in real time.
- [ ] Robot halts movement within a reasonable distance after recognizing the sign.
- [ ] OCR pipeline works reliably under normal lighting conditions.
- [ ] System is documented clearly enough that a classmate could replicate the setup using the README.
