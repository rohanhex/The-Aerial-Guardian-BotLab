**The Aerial Guardian**:
Advanced Small-Object Detection & Tracking for UAV Platforms

## 🎬 Live Demo
▶️ [Watch the Final Demo — Phase 3 Output (780 frames)](https://github.com/rohanhex/The-Aerial-Guardian-BotLab/releases/tag/v1.0)


Project Summary: This repository contains a specialised computer vision pipeline developed for the BotLab Dynamics practical assignment. The system is designed to tacklethe three primary hurdles of aerial imagery - High Altitude, Small Target Size, and Significant Camera Motion.

**Architecture & Engineering Decisions [Technical Task 1]:** For this challenge, I selected YOLOv8-Nano as the base architecture.

i. Choice logic - At only ~6-12MB, it fits well within the 300MB constraints. Its architectural efficiency allows for high-inference speed necessary for real time drone operations.

ii. Small Object Handling - To compensate the loss of resolution at high altitudes, I have opted for 1920p instead of default 680p resolution. This ensures that even 10-pixels target has enough spatial information for model to recognize.

iii. Engineering Trafe-offs: I have prioritized the inference speed over mAP. In drone context, a slightly lower precision is acceptable if the FPS doesn't drop below 24, allowing the tracker to consistenty maintain the ID stability through high resolution.

**Ego-Motion & ID Stability [Technical Task 2]:** Maintaining consistent IDs while the camera is moving is the core challenge of this assignment.

The Problem: As camera moves continously, ID switching occurs, causing the background to move and this confuses the tracker. 

The solution: I have implemented Optical-Flow based Ego-motion Compensation. The background shift (due to drone motion) is calculated using the Farneback method and the pipeine stabilizes the tracking in the virtual ground-plane.
Also for this , the tracker is set to BoT-SORT, as it integrates camera motion compensation more effectively than the standard DeepSORT.

**Advanced Tracking Logic: Occlusions & Stability [Technical Task 2]:** Aerial surveillance is prone to frequent occlusions - targets disappearing behind buildings, trees or underpasses. To maintain ID consistency during these blind spots, the system is utilizing a Temporal Buffer & Euclidean Prediction approach.

i. Persistence Logic: I tuned the   tracker = 'botsort.yaml' parameter to maintain a memory buffer of 24/30 frames. If a car/person disappears, the tracker doesn't delete the ID immediately but holds the track state in "lost pool".

ii. Spatial Matching: When a detection re-appears, the tracker calculated the Euclidean distance and feature similarity between the new detection and the "lost" tracks. If match found, system re-assigns the previous ID instead of a new one.

iii. Engineering Trade-off: Increasing the memory buffer can lead to ID leaking and so the buffer for lost tracking is set to 24/30 frames only. Also the "lost tracking" is for 48/60 frames and if the target re-appears after being lost for more, it is assigned new ID. In case where the target re-appears within buffer time, to prevent temporal jump the system keeps only the latest coordinates in the memory.

**Memory Management & Resource Optimisation [Technical task 3]:** I have implemented following memory optimization strategies:

i. Generator-based Processing: Instead of loading the entire video into RAM, I used a Frame-by-Frame Generator. This ensures the memory requirement remains constant regardless of video size ie. constant space complexity.

ii. VRAM Optimisation: By setting image = 1920p, specific memmory-aligned resolution is used so that it leaves enough information for tracker to track better.

iii. Precision Quantization: While developed in FP32 for this assignment, the architecture is designed to support INT8 Quantization. This would reduce the model size by ~75% and memory bandwidth by half. This would suite the NVIDIA Jetson architecture. 

iv. Automatic Garbage Collection: I have implemented explicit clearing of the track history of targets that have been lost for 48/60 frames, preventing the memory leaks during long duration surveillance. 


**Performance Analysis & Sensitivity:** As per requirement, the hardware specific performance and the sensitivity study of the 'noise' vs 'tracking' trade-off is below:

| Configuration | Confidence/IOU Params | Precision | Recall | F1 Score (Est.) |
| :--- | :--- | :--- | :--- | :--- |
| **Strict** | `0.5 < conf < 0.7`, `iou = 0.7` | 59% - 64% | 18% - 20% | ~28% |
| **Balanced** | `0.15 < conf < 0.35`, `iou = 0.3` | 53% - 57% | 26% - 28% | **~30%** |

For our assignment I have chosen the low configuration where conf = 0.2, the precision comes out to be 57.07% while recall is 28.51% thereby making F1 score ~ 30%. This accepts a higher 'noise' (slight drop in precision)to ensure maximum situational awaeness, which is vital for search and rescue operations. 
I chose the Balanced configuration because in drone-based search or security, missing a target (Low Recall) is often a bigger failure than dealing with occasional false detections (Low Precision).

**Hardware: NVIDIA T4 x2 GPU [Kaggle Platform]
Inference Speed: 30 FPS**


**Development Roadmap: System Design**
The system was developed in four distinct phases, prioritising stability and mathematical validation at each step.

Phase 0: The Baseline [Detection] - Establish a working inference pipeline on the VisDrone Sequence. 

Successfully loaded YOLOv8-Nano and verified target detection for the 'pedestrian' and 'vehicle' classes. 

Phase 1.1: Implementing Tracking for 'class=0'


Phase 1.2: Robust Tracking & Memory Management - Maintaining Consistent ID and stay within Hardware limits

Tuned BoT-SORT parameters & implemented a Temporal buffers (30 frames) to handle the occlusions. Developed a Frame-Generator architecture to process the 780-frame sequence with constant memory footprint.

Phase 2.1: Virtual-Fencing & Spatial Analytics - Defining Restricted zones 

A 'No-Pedestrian Zone' is intoduced as part of surveillance in the frame. OpenCV pointPolygon test is used to identify how many IDs have violated the zone by tracking their feet coordinates. A counter keeps track of such violations.

As drone drift happens, the restricted zone also moves in the frame due to Camera-Ego motion. This also records the violations which are not actually true.

Phase 2.2: Addressing the Camera-Ego Motion 

Linked the virtual fencing to Phase 1.1 drift vectors. This ensured the restricted zone doesn't drift when drone drifts. The zone is locked to actual physical coordiantes. The violations are corrected.
Introduced multiple small restricted zones instead of a large one.

Phase 3: Velocity Analytics & Sensitivity Study - Extracting the actionalble speed data and validate system performance.

Implemented a Temporal Velocity Engine (kmph) using a 1-sec rolling window and pixel-to-meter scaling.

Conducted the sensitivity analysis for case 1 where parameters are stictly constrained and case 2 where parameters are more relaxed and explained the different use case scenario maintaining a balance. 


**Future Optimisation: Implementing SAHI**

To push system's recall value beyond 28.4%, we need to implement SAHI [Slicing Aided Hyper Inference] to generate a clearer image in row resolution ie 640p.

While SAHI would significantly boost recall score, it increases the number of inference per frame thereby reducing the inference speed.
