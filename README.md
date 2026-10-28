# Urban Sense AI

Urban Sense AI is a collection of computer-vision prototypes for analyzing
street and public-space footage. It contains independent command-line tools
for pothole detection, crowd and traffic monitoring, and accident alerts.

The detectors are run separately from their own directories. There is no
root-level Python package or shared dependency file.

## Detectors

| Tool | Location | What it does |
|---|---|---|
| Pothole detector | `models/Pothole detector V1/` | Detects potholes in road images and provides webcam/video CLI options. Uses the bundled trained YOLO weights. |
| Crowd and traffic detector | `models/CrowdnTraffic detector V2/` | Counts people and vehicles with YOLO, or estimates dense crowd regions with CSRNet. Supports images, video, and camera streams. |
| Accident detector | `models/Accident detector V2/` | Tracks vehicles and scores motion anomalies to generate possible accident alerts. An optional trained classifier can confirm alerts. |

These are research/prototype systems. Detection results can be inaccurate and
should not be treated as a substitute for human review or safety-critical
systems.

## Requirements

- Python 3.10 or later is recommended.
- Install dependencies separately for the detector you want to use.
- A webcam or video source is needed for live/video processing; image modes
  only need an image.
- A CUDA-capable GPU is optional. CPU inference is supported, though slower.

Create and activate a virtual environment from the project root in
PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell prevents script activation, activate the environment from
Command Prompt with `.venv\Scripts\activate.bat`.

### Pothole detector dependencies

```powershell
python -m pip install ultralytics opencv-python
```

### Crowd and traffic detector dependencies

```powershell
Set-Location "models\CrowdnTraffic detector V2"
python -m pip install -r requirements.txt
```

The requirements file installs Ultralytics, OpenCV, NumPy, PyTorch, and
Torchvision. Use the [PyTorch installation selector](https://pytorch.org/get-started/locally/)
if you need a CUDA-specific PyTorch build.

### Accident detector dependencies

```powershell
python -m pip install ultralytics opencv-python numpy
```

## Usage

Run commands from the corresponding detector directory. This keeps relative
model and output paths pointed at the expected files.

### Pothole detection

```powershell
Set-Location "models\Pothole detector V1"

# Analyze an image; writes the annotated image to output.jpg by default
python cli.py --image road.jpg

# Use a webcam
python cli.py --webcam
```

The bundled model is loaded from
`runs/detect/pothole_yolov8/weights/best.pt`. The image, video, and webcam
options are defined in `cli.py`; video processing may require fixes in the
current CLI implementation before it works.

### Crowd and traffic detection with YOLO

```powershell
Set-Location "models\CrowdnTraffic detector V2"

# Count people in an image
python detector.py --source crowd.jpg --mode crowd --save

# Count vehicles in a video and save annotated output
python detector.py --source traffic.mp4 --mode traffic --save --output result.mp4

# Count people and vehicles from the default webcam
python detector.py --source 0 --mode both
```

Modes are `crowd` (people), `traffic` (vehicles), and `both`. Use
`--help` to see additional options, including confidence thresholds,
frame-skipping, and headless operation (`--no-show`). YOLO weights are
downloaded automatically on first use if they are not already available.

### Crowd-density estimation with CSRNet

CSRNet estimates a density heatmap rather than drawing an individual box for
each person. This is intended for crowded scenes where people overlap.

```powershell
Set-Location "models\CrowdnTraffic detector V2"

# Analyze an image and save its annotated output
python csrnet_detector.py --source image.jpg --weights weights\csrnet_shanghaiA.pth --save

# Analyze a video
python csrnet_detector.py --source video.mp4 --weights weights\csrnet_shanghaiA.pth --save --output density.mp4

# Run the CSRNet engine through the unified entry point
python run.py csrnet --source video.mp4 --weights weights\csrnet_shanghaiA.pth --save
```

The repository includes `weights/csrnet_shanghaiA.pth`. If you replace it or
it is missing, obtain a compatible pretrained checkpoint from the
[CSRNet-pytorch project](https://github.com/leeyeehoo/CSRNet-pytorch).
Common pretrained checkpoints estimate **people** density; they are not
trained for vehicle density.

### Accident alerts

```powershell
Set-Location "models\Accident detector V2"

# Analyze a video; show the annotated footage
python trajectory_accident_detector.py --source video.mp4 --display

# Use a webcam
python trajectory_accident_detector.py --source 0 --display

# Optionally use a trained accident classifier as a confirmation filter
python trajectory_accident_detector.py --source video.mp4 `
  --classifier-weights runs\detect\accident_detection_model-5\weights\best.pt `
  --display
```

By default, the vehicle detector uses `yolov8n.pt`. The trajectory-based
detector combines movement, deceleration, bounding-box changes, and vehicle
overlap to flag anomalies. Alerts and clips are saved under
`accident_alerts\`, including `trajectory_alerts_log.csv`. Adjust alert
thresholds and output location with the options shown by:

```powershell
python trajectory_accident_detector.py --help
```

An alternative `detect_accidents.py` script detects a model class whose name
contains `accident`; it requires a compatible trained detector checkpoint:

```powershell
python detect_accidents.py --source video.mp4 --weights path\to\best.pt
```

## Project layout

```text
api/
  main.py                         # API scaffold; no endpoints are implemented yet
models/
  Accident detector V2/           # Trajectory-based accident alert scripts and weights
  CrowdnTraffic detector V2/       # YOLO and CSRNet crowd/traffic tools
  Pothole detector V1/             # Pothole detector, CLI, and trained weights
```

## Notes

- Paths in the examples are relative to each detector's directory.
- For video windows, press `q` to quit when the detector supports display.
- Performance and accuracy depend on the input, camera angle, hardware, and
  model weights. Tune thresholds for the deployment environment.
- The `api/main.py` file currently contains imports only; the detectors are
  presently used through their command-line scripts.
