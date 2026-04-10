\# Follow the below steps to run the code:



\# 1. clone the repository:

\########## Open your terminal (or Command Prompt) and run:

*git clone* [*https://github.com/rohanhex/The-Aerial-Guardian-BotLab.git*](https://github.com/rohanhex/The-Aerial-Guardian-BotLab.git)

*cd The-Aerial-Guardian-BotLab*



\# 2. Install dependencies from the requirements text document

*pip install -r requirements.txt*



\# 3. Dataset Preparation

\##############The Aerial Guardian is optimized for the VisDrone-MOT dataset.

&#x20;             Download the uav0000077\_00720\_v image sequence.

&#x20;             Create a data/ directory in the project root.

&#x20;             Extract the images into: data/uav0000077\_00720\_v/ 

\###########################################################################

link for dataset:   [https://github.com/VisDrone/VisDrone-Dataset](https://github.com/VisDrone/VisDrone-Dataset)

&#x20;                   [https://drive.google.com/file/d/1rqnKe9IgU\_crMaxRoel9\_nuUsMEBBVQu/view?usp=sharing](https://drive.google.com/file/d/1rqnKe9IgU_crMaxRoel9_nuUsMEBBVQu/view?usp=sharing)

&#x20;                   

\###########################################################################

The Folder structure should look like this:

Aerial-Guardian/

├── main.py

├── data/

│   └── uav0000077\_00720\_v/

│       ├── 0000001.jpg

│       ├── ...

│       └── 0000780.jpg



\# 4. Run Final Production Engine (Phase 3)

\########## 

This will process the sequence, apply drift correction, calculate velocities, and export the final annotated video.

*python main.py*



\##########

To see the development logic (e.g., Geofencing without Velocity), run the versioned scripts:

python versions/main\_phase2.py



\#################################################################################################

\# 5. Hardware Considerations

GPU (NVIDIA): If a CUDA-enabled GPU is detected, the pipeline will automatically utilize it for \~30+ FPS inference.

CPU: The system will run on standard CPUs but at a reduced frame rate (\~3-5 FPS). The output accuracy remains identical regardless of hardware.





