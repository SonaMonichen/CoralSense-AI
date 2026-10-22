<div align="center">
CoralSense AI
An AI-integrated drifting buoy for geo-referenced coral reef health assessment
Underwater imaging · in-situ water sensing · U-Net semantic segmentation · GPS + sensor data fusion
![Python](#tech-stack)
![PyTorch](#tech-stack)
![OpenCV](#tech-stack)
![Status](#project-status)
![OCEANAI-2026](docs/CoralSenseAI_OCEANAI2026_Slides.pdf)
Abstract · Slides · Literature Review
</div>
<p align="center">
  <img src="docs/images/pipeline-illustration.jpg" alt="Illustrative CoralSense AI pipeline: raw image, enhanced image, coral segmentation, health map" width="760">
  <br>
  <sub><i>Illustration of the intended pipeline. These images are <b>not</b> model output.</i></sub>
</p>
> \[!NOTE]
> \*\*Status:\*\* In development. The architecture, methodology and dataset plan are complete, and the AI pipeline is being implemented. Buoy hardware, field deployment and final evaluation are still ahead, so this repository reports no accuracy figures or field results yet.
Contents
Overview · System Architecture · AI Pipeline · Tech Stack · Getting Started · Evaluation · Project Status · Team
---
Overview
Coral reefs cover less than 1% of the ocean floor but support about 25% of marine life. Heat stress makes corals expel their symbiotic algae and turn white. Bleached coral can recover, but prolonged stress kills it, so monitoring has to show where bleaching happens, how much of the reef is affected, and under what water conditions.
Current tools each cover only part of this. Fixed buoys such as NOAA CREWS measure water conditions but don't move or capture images. AUVs move and capture images but are expensive and complex to operate. AI tools such as CoralNet analyse images but depend on a separate system to collect them, with no environmental context.
CoralSense AI combines all three in one low-cost platform. A buoy drifts across the reef with the current and records:
coral images
water conditions
position
Each image is then turned into a geo-referenced reef-health record.
Engineering highlights
Pixel-level health mapping: a U-Net labels each coral pixel as Healthy, Bleached or Dead, and reports % cover per class.
Multi-sensor fusion: each image is matched by timestamp to a GPS fix and to temperature, pH and salinity readings.
Explicit position uncertainty: satellite signals don't reach underwater, so the GPS fix comes from the surface unit. The offset from tether length and drift is reported with each record instead of being ignored.
Plans for domain shift: the training data and the target reefs differ, and the plan validates on local expert-labelled imagery before deployment.
---
System Architecture
<p align="center">
  <img src="docs/images/architecture.png" alt="Eight-stage CoralSense AI architecture from buoy deployment to geo-referenced reef-health record" width="900">
</p>
<table>
<tr>
<td width="44%"><img src="docs/images/system-schematic.png" alt="Conceptual schematic: surface buoy with GPS and battery, tethered underwater module with camera and sensors, ballast"></td>
<td>
Hardware concept (specifications under development)
Current-drifting: ocean currents carry the buoy; there's no propulsion.
Battery-powered: the buoy runs on its own battery, with no solar or shore power.
Surface unit: GPS, battery, controller and data logger.
Underwater module: a downward camera plus temperature, pH and salinity sensors.
Local logging: data is stored onboard and retrieved after deployment. There is no real-time telemetry, by design.
Modular build: reusable, repairable, readily available components for low-cost repeat deployments.
</td>
</tr>
</table>
---
AI Pipeline
Stage	What it does
1. Preprocessing	Corrects underwater colour distortion (red light is absorbed first), contrast loss and poor visibility
2. Segmentation	A U-Net (Ronneberger et al., 2015) encoder–decoder with skip connections assigns a class to every pixel
3. Health output	Labels coral regions Healthy / Bleached / Dead plus background, and computes % cover per class
4. Data fusion	Joins each mask with the nearest GPS fix and sensor reading on a common clock
Dataset: Coralscapes (Sauder et al., 2025) has 2,075 expert-annotated images across 39 benthic classes from 35 Red Sea sites. The plan maps its coral annotations to the three target classes by label meaning:
Coralscapes label	Target class
Coral, "… Alive"	Healthy*
Coral, "… Bleached"	Bleached
Coral, "… Dead"	Dead
Non-coral (sand, rubble, fish, …)	Background
<sub>* "Alive" doesn't automatically mean "healthy", so the mapping needs expert review. Coralscapes is Red Sea diver imagery, while the target is buoy imagery of Indian reefs, so local labelled data is needed before deployment.</sub>
Output record (one per image): `timestamp` · `position ± uncertainty` · `temperature` · `pH` · `salinity` · `% healthy` · `% bleached` · `% dead`
---
Tech Stack
	
Language	Python
Deep learning	PyTorch, U-Net
Computer vision	OpenCV
Data & analysis	NumPy, Pandas, Matplotlib, scikit-learn
Environment	Jupyter Notebook, Google Colab
Dataset	Coralscapes
Hardware (planned)	Raspberry Pi, camera, temperature / pH / salinity sensors, GPS
---
Getting Started
```bash
git clone https://github.com/<your-username>/CoralSense-AI.git
cd CoralSense-AI
pip install torch torchvision opencv-python numpy pandas matplotlib scikit-learn jupyter
jupyter notebook
```
Open `CoralSense\_AI.ipynb` and set the dataset path for your machine or Colab session.
Run the cells in order.
To run inference on the files in `test\_samples/`, use `Test\_Model\_CoralSense.ipynb`. The trained checkpoint `best\_model.pt` isn't stored in this repo; place it at the path the notebook expects.
```
CoralSense-AI/
├── CoralSense\_AI.ipynb           # Main project notebook
├── Test\_Model\_CoralSense.ipynb   # Model-testing notebook
├── class\_names.txt               # Class labels
├── test\_samples/                 # Sample images
└── docs/
    ├── images/                   # README figures
    ├── CoralSenseAI\_Abstract.pdf
    ├── CoralSenseAI\_OCEANAI2026\_Slides.pdf
    └── CoralSenseAI\_Literature\_Review.pdf
```
---
Evaluation
Results haven't been reported yet. The plan below will be completed before any field deployment.
Component	Planned metrics
Segmentation	Per-class IoU, mean IoU, pixel accuracy on held-out, expert-labelled subsets
Health classification	Precision, recall, F1, confusion matrix; with vs. without preprocessing
Hardware (bench)	Sensor calibration, waterproofing, camera–GPS–sensor time sync
<!--
  ADD WHEN AVAILABLE (only real, reproducible numbers):

  ### Results
  | Class | IoU |
  |---|---|
  | Healthy | 0.xx |
  | Bleached | 0.xx |
  | Dead | 0.xx |
  | Background | 0.xx |
  | \*\*Mean IoU\*\* | \*\*0.xx\*\* |
  Pixel accuracy: 0.xx · Split: <train/val/test sizes> · Epochs: <n> · Hardware: <GPU>

  <p align="center"><img src="docs/images/sample-prediction.png" width="760"></p>
  <sub>Input image | ground truth | predicted mask on a held-out Coralscapes image.</sub>

\---

## Project Status

|Completed|Next|Not yet done|
|-|-|-|
|Literature review \& gap analysis|Preprocessing + U-Net implementation|Field deployment|
|System architecture|Hardware build \& sensor integration|Final accuracy figures|
|AI methodology|System integration|Hardware validation|
|Dataset planning \& class mapping|Evaluation|Real-world results|

<details>
<summary><b>Known challenges and planned mitigations</b></summary>
<br>

|Challenge|Planned mitigation (not yet validated)|
|-|-|
|Drift path depends on currents|Plan deployments around local currents and tides; boat retrieval|
|Turbidity, lighting, motion blur|Preprocessing; discard low-quality frames; ballast for stability|
|Domain shift (Red Sea → Indian reefs; diver → buoy view)|Local expert-labelled imagery before deployment|
|Sensor calibration and biofouling|Calibrate before and after each deployment; short deployments|
|GPS-to-module offset|Characterise the offset; report uncertainty per record|
|Battery endurance|Duty-cycled capture; size the battery during hardware development|

</details>

**Future scope:** validate the class mapping with marine experts → benchmark the pipeline → bench-test the buoy → collect labelled imagery from Indian reefs → run field surveys as a low-cost complement to fixed-buoy and AUV monitoring.

\---

## Team

**Sona Monichen** · **Milan Mathew** · **Mohammed Syed Marjuk**
Guided by **Dr. Anupama Jims** ([anupama.jims@cvv.ac.in](mailto:anupama.jims@cvv.ac.in))
Department of Computer Science, Chinmaya Vishwavidyapeeth (Deemed to be University), Ernakulam, Kerala, India

Presented as an oral talk at **OCEANAI-2026**, Sathyabama Research Park, 28–29 September 2026.

<!--
  RECOMMENDED for a personal portfolio fork: add a short "My contributions" section, e.g.
  - Implemented ... in PyTorch
  - Built the preprocessing step using ...
  - Wrote the Coralscapes label-mapping script ...

<details>
<summary><b>References</b></summary>
<br>

1. O. Ronneberger, P. Fischer, T. Brox, "U-Net: Convolutional Networks for Biomedical Image Segmentation," *MICCAI*, 2015.
2. J. Sauder et al., "The Coralscapes Dataset: Semantic Scene Understanding in Coral Reefs," arXiv:2503.20000, 2025.
3. M. Xanthidis, A. Quattrini Li, I. Rekleitis, "Shallow coral reef surveying by inexpensive drifters," *IEEE OCEANS*, 2016.
4. NOAA, Coral Reef Early Warning System (CREWS), NOAA Coral Reef Conservation Program.
5. O. Beijbom et al., "Towards Automated Annotation of Benthic Survey Images," *PLoS ONE*, 2015 (CoralNet).
6. E. T. Raber, S. R. Schill, "Reef Rover: A Low-Cost Small Autonomous USV for Mapping and Monitoring Coral Reefs," *Drones*, 3(2):38, 2019.
7. NOAA National Ocean Service, ["What is coral bleaching?"](https://oceanservice.noaa.gov/facts/coral_bleach.html)

</details>
