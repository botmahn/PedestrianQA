# PedestrianQA: A Benchmark for Vision-Language Models on Pedestrian Intention and Trajectory Prediction

<p align="center">
  <img src="assets/pedestrianqa_teaser.png" alt="PedestrianQA teaser showing observation frames, pedestrian intention prediction, trajectory prediction, and rationale generation." width="100%">
</p>

<p align="center">
  <a href="https://botmahn.github.io/pqa_pw/assets/pedestrianqa_camera_ready.pdf"><img src="https://img.shields.io/badge/Paper-ICRA%202026-blue" alt="Paper"></a>
  <a href="https://huggingface.co/collections/namansmishaps/pedestrianqa"><img src="https://img.shields.io/badge/Models-Hugging%20Face-yellow" alt="Models on HuggingFace"></a>
  <a href="https://botmahn.github.io/pqa_pw/"><img src="https://img.shields.io/badge/Project-Webpage-green" alt="Project Webpage"></a>
</p>

This repository contains the official PedestrianQA dataset. PedestrianQA is a video question-answering benchmark for pedestrian behavior understanding in autonomous driving. It reformulates two safety-critical tasks as multimodal QA:

- **Pedestrian Intention Prediction (PIP):** predict whether a target pedestrian will cross the road.
- **Pedestrian Trajectory Prediction (PTP):** predict the target pedestrian's future bounding-box trajectory.

Each QA sample includes structured natural-language rationales that explain the prediction using spatial, temporal, mathematical, ego-vehicle, and scene-context cues. The benchmark is built from four pedestrian datasets: **IDD-PeD**, **JAAD**, **PIE**, and **TITAN**.

## Repository Contents

This repository contains PedestrianQA annotation files and CSV indexes. Raw video frames/videos are not included; use the source dataset names, video IDs, pedestrian IDs, and frame ranges in the CSV files to align these annotations with the original datasets.

```text
.
+-- annotations/
|   +-- iddped/
|   +-- jaad/
|   +-- pie/
|   +-- titan/
+-- csv/
    +-- iddped.csv
    +-- jaad.csv
    +-- pie.csv
    +-- titan.csv
```

## Dataset Summary

Every row in `csv/<dataset>.csv` maps to exactly one JSON file at `annotations/<dataset>/<uuid>.json`.

| Source | Rows / JSON files | Train | Test | PIP QAs | PTP QAs |
| --- | ---: | ---: | ---: | ---: | ---: |
| iddped | 3,966 | 2,611 | 1,355 | 3,966 | 199 |
| jaad | 356 | 179 | 177 | 356 | 280 |
| pie | 1,595 | 878 | 717 | 1,595 | 788 |
| titan | 4,800 | 3,891 | 909 | 4,800 | 2,327 |
| **Total** | **10,717** | **7,559** | **3,158** | **10,717** | **3,594** |

## CSV Schema

Each CSV file has the same columns:

| Column | Description |
| --- | --- |
| `uuid` | Unique sample identifier. Also the JSON filename. |
| `set` | Source dataset set/partition identifier, when available. |
| `video` | Source video or clip filename. |
| `pedestrian_id` | Target pedestrian identifier in the source dataset. |
| `start_frame` | First observed frame for the QA sample. |
| `end_frame` | Last observed frame for the QA sample. |
| `split` | `train` or `test`. |

## Annotation Schema

Each JSON file contains a `pedestrian_crossing_intention_qa` object. Some files additionally contain `pedestrian_trajectory_prediction_qa`.

### Intention QA

```json
{
  "pedestrian_crossing_intention_qa": {
    "Question": "Will the pedestrian located at [x1, y1, x2, y2] in frame 1 cross the road?...",
    "Answer": "Yes",
    "Spatial_Reason": "...",
    "Temporal_Reason": "...",
    "Mathematical_Reason": "...",
    "Ego_Vehicle_Reason": "...",
    "Scene_Context_Reason": "...",
    "Conclusion": "..."
  }
}
```

The `Answer` is a binary crossing-intention label: `Yes` or `No`.

### Trajectory QA

```json
{
  "pedestrian_trajectory_prediction_qa": {
    "Question": "Given the trajectory of the pedestrian located at ... predict their trajectory...",
    "Answer": [[x1, y1, x2, y2], "..."],
    "Spatial_Reason": "...",
    "Temporal_Reason": "...",
    "Mathematical_Reason": "...",
    "Ego_Vehicle_Reason": "...",
    "Scene_Context_Reason": "...",
    "Final_Destination": "...",
    "Conclusion": "..."
  }
}
```

Bounding boxes use image pixel coordinates in `[x1, y1, x2, y2]` format.

For JAAD, PIE, and IDD-PeD, the paper samples 15 observation frames, corresponding to 0.5 seconds at approximately 30 FPS. Trajectory samples predict the next 45 frames. For TITAN, the paper samples 10 observation frames, corresponding to 1 second at 10 FPS, and predicts the next 20 frames.

## Loading the Dataset

```python
import csv
import json
from pathlib import Path

root = Path(".")
dataset = "pie"

with open(root / "csv" / f"{dataset}.csv") as f:
    rows = list(csv.DictReader(f))

sample = rows[0]
annotation_path = root / "annotations" / dataset / f"{sample['uuid']}.json"

with open(annotation_path) as f:
    annotation = json.load(f)

pip_qa = annotation["pedestrian_crossing_intention_qa"]
ptp_qa = annotation.get("pedestrian_trajectory_prediction_qa")

print(sample["video"], sample["pedestrian_id"], sample["start_frame"], sample["end_frame"])
print(pip_qa["Question"])
print(pip_qa["Answer"])
```

## Data Access

PedestrianQA is derived from IDD-PeD, JAAD, PIE, and TITAN. To reconstruct full video inputs, acquire all necessary licenses and permissions before downloading the original datasets from their official sources:

- PIE: https://data.nvision2.eecs.yorku.ca/PIE_dataset/
- JAAD: https://data.nvision2.eecs.yorku.ca/JAAD_dataset/
- TITAN: https://usa.honda-ri.com/titan
- IDD-PeD: https://cvit.iiit.ac.in/research/projects/cvit-projects/iddped

After downloading the source datasets, use the CSV metadata to locate the target video, pedestrian, and frame interval. Respect the licenses and terms of use of the original source datasets. No standalone license file is currently included in this repository.

## Citation

If this dataset is useful for your work, please cite:

```bibtex
@inproceedings{mishra2026pedestrianqa,
  title = {PedestrianQA: A Benchmark for Vision-Language Models on Pedestrian Intention and Trajectory Prediction},
  author = {Mishra, Naman and Gangisetty, Shankar and Jawahar, C. V.},
  booktitle = {Proceedings of the IEEE International Conference on Robotics and Automation (ICRA)},
  year = {2026},
  url = {https://github.com/botmahn/PedestrianQA}
}
```
