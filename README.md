**The Aerial Guardian**:
Advanced Small-Object Detection & Tracking for UAV Platforms

Project Summary: This repository contains a specialised computer vision pipeline developed for the BotLab Dynamics practical assignment. The system is designed to tacklethe three primary hurdles of aerial imagery - High Altitude, Small Target Size, and Significant Camera Motion.

**Architecture & Engineering Decisions [Technical Task 1]:** For this challenge, I selected YOLOv8-Nano as the base architecture.

i. Choice logic - At only ~6-12MB, it fits well within the 300MB constraints. Its architectural efficiency allows for high-inference speed necessary for real time drone operations.

ii. Small Object Handling - To compensate the loss of resolution at high altitudes, I have opted for 1920p instead of default 680p resolution. This ensures that even 10-pixels target has enough spatial information for model to recognize.

iii. Engineering Trafe-offs: I have prioritized the inference speed over mAP. In drone context, a slightly lower precision is acceptable if the FPS doesn't drop below 24, allowing the tracker to consistenty maintain the ID stability through high resolution.

**Ego-Motion & ID Stability [Technical Task 2]:** Maintaining consistent IDs while the camera is moving is the core challenge of this assignment.

The Problem: As camera moves continously, ID switching occurs, causing the background to move and this confuses the tracker. 

The solution: I have implemented Optical-Flow based Ego-motion Compensation. The background shift (due to drone motion) is calculated using the Farneback method and the pipeine stabilizes the tracking in the virtual ground-plane.
Also for this , the tracker is set to BoT-SORT, as it integrates camera motion compensation more effectively than the standard DeepSORT.
