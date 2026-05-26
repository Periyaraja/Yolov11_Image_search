# YOLOv11 Computer Vision Image Search

A Streamlit application that turns object detection results into a searchable image gallery. The app runs YOLOv11 on a folder of images, saves the detections as JSON metadata, and lets you search for images by detected object classes such as `person`, `car`, `apple`, `bed`, or `baseball bat`.

The project is built for quick computer vision experiments, dataset exploration, and demos where you need to answer questions like:

- Which images contain a bed?
- Which images contain either an apple or a baseball bat?
- Which images contain all selected objects?
- Which images contain a selected object no more than a chosen number of times?

## Screenshots

### Home screen

![Home screen](docs/screenshots/app-home.png)

### Search filters

![Search filters](docs/screenshots/search-ui.png)

### Results overview

![Results overview](docs/screenshots/results-overview.png)

### Detection results

![Detection results](docs/screenshots/results-grid.png)

## Features

- Process a complete image directory with YOLOv11 object detection.
- Load existing metadata instead of running inference every time.
- Search by one or more detected object classes.
- Use `OR` mode to match any selected class.
- Use `AND` mode to require every selected class.
- Add optional maximum count thresholds per selected class.
- Display matching images in a responsive grid.
- Draw bounding boxes and confidence labels on detected objects.
- Highlight only the classes used in the current search.
- Change the number of result columns from the UI.
- Export the current search results as JSON.
- Run on CPU or GPU depending on the installed PyTorch setup.

## Tech Stack

- Python
- Streamlit
- Ultralytics YOLO
- PyTorch
- Pillow
- PyYAML
- Pandas and NumPy
- OpenCV headless

## Project Structure

```text
Yolov11_Image_search/
|-- app.py
|-- requirements.txt
|-- yolo11m.pt
|-- configs/
|   `-- default.yaml
|-- src/
|   |-- config.py
|   |-- inference.py
|   `-- utils.py
|-- coco-val-2017-500/
|   `-- sample image dataset
|-- data/
|   `-- processed/
|       `-- coco-val-2017-500/
|           `-- metadata.json
|-- docs/
|   `-- screenshots/
|-- Demo/
|   `-- demo.py
`-- test/
    `-- streamlit_basics.py
```

### Important Files

| File | Purpose |
| --- | --- |
| `app.py` | Main Streamlit application. Handles UI, searching, result display, bounding boxes, and JSON export. |
| `src/inference.py` | Wraps Ultralytics YOLO inference and converts model predictions into metadata records. |
| `src/utils.py` | Saves metadata, loads metadata, and builds class/count filter options. |
| `src/config.py` | Loads and saves YAML configuration. |
| `configs/default.yaml` | Stores model confidence threshold and supported image extensions. |
| `data/processed/coco-val-2017-500/metadata.json` | Ready-to-load metadata for the included sample image folder. |
| `coco-val-2017-500/` | Included sample image dataset used for demos. |
| `yolo11m.pt` | YOLOv11 model weights used by default. |

## Requirements

Recommended setup:

- Python 3.11
- Windows, macOS, or Linux
- At least 4 GB RAM for small demos
- More RAM and a GPU for larger image folders
- Optional NVIDIA GPU with a CUDA-compatible PyTorch install

CPU works fine for the included 500 image demo, but GPU inference is much faster for larger datasets.

## Installation

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd Yolov11_Image_search
```

### 2. Create and activate a virtual environment

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\activate
```

macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PyTorch does not install correctly for your machine, install PyTorch from the official selector for your CPU or CUDA version first, then run `pip install -r requirements.txt` again.

## Conda Setup

If you prefer Conda, use one of these setups.

### CPU environment

```bash
conda create -n yolo_image_search python=3.11 -y
conda activate yolo_image_search
pip install -r requirements.txt
```

### GPU environment

```bash
conda create -n yolo_image_search_gpu python=3.11 -y
conda activate yolo_image_search_gpu
conda install pytorch==2.5.1 torchvision==0.20.1 pytorch-cuda=12.4 -c pytorch -c nvidia
pip install -r requirements.txt
```

Check whether PyTorch can see your GPU:

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

If it prints `True`, the app should use GPU automatically.

## Running the App

Start Streamlit from the project root:

```bash
streamlit run app.py
```

Streamlit usually opens this URL automatically:

```text
http://localhost:8501
```

To use a custom port:

```bash
streamlit run app.py --server.port 8080
```

## Fast Demo With Included Metadata

Use this path if you want to try the app immediately without re-running YOLO inference:

```text
data/processed/coco-val-2017-500/metadata.json
```

Steps:

1. Run the app with `streamlit run app.py`.
2. Select `Load existing metadata`.
3. Paste `data/processed/coco-val-2017-500/metadata.json`.
4. Click `Load Metadata`.
5. In `Classes to search for`, select classes such as `apple`, `baseball bat`, and `bed`.
6. Keep search mode as `Any of selected classes (OR)`.
7. Click `Search Images`.

With the included sample metadata, the `apple`, `baseball bat`, and `bed` example returns 21 matching images.

## Processing Your Own Images

Use this when you want to create fresh metadata for a new image folder.

1. Put your images in a folder.
2. Run the app.
3. Select `Process new images`.
4. Enter the image folder path.
5. Enter the model weights path, usually:

```text
yolo11m.pt
```

6. Click `Start Inference`.
7. Wait for processing to complete.
8. The app shows where the new `metadata.json` file was saved.
9. Use the search panel that appears after processing.

Supported image extensions are configured in `configs/default.yaml`:

```yaml
data:
  image_extension: [".jpg", ".jpeg", ".png"]
```

## Recommended Data Layout

For repeatable projects, keep raw and processed data in a simple structure:

```text
data/
|-- raw/
|   `-- my-images/
|       |-- image_001.jpg
|       `-- image_002.jpg
`-- processed/
    `-- my-images/
        `-- metadata.json
```

Then process:

```text
data/raw/my-images
```

The app saves metadata under the matching processed folder.

## How the App Works

### 1. Inference

`src/inference.py` loads the YOLO model and runs detection on each supported image file:

```python
inferencer = YOLOv11Inference("yolo11m.pt")
metadata = inferencer.process_directory("path/to/images")
```

Each image becomes one metadata record with:

- `image_path`: path to the source image
- `detections`: list of detected objects
- `class`: detected object name
- `confidence`: model confidence score
- `bbox`: bounding box coordinates in `[x1, y1, x2, y2]` format
- `total_objects`: total detections in the image
- `unique_class`: unique detected classes
- `class_counts`: count of each detected class

Example metadata shape:

```json
{
  "image_path": "coco-val-2017-500/000000000139.jpg",
  "detections": [
    {
      "class": "person",
      "confidence": 0.83,
      "bbox": [407.7, 157.1, 464.2, 297.0],
      "count": 1
    }
  ],
  "total_objects": 1,
  "unique_class": ["person"],
  "class_counts": {
    "person": 1
  }
}
```

### 2. Metadata Loading

`src/utils.py` loads the JSON metadata and extracts:

- every unique class available in the dataset
- possible object count options for each class

That is how the class dropdown and count threshold dropdowns are built.

### 3. Search

Search behavior is controlled by the selected mode:

| Mode | Meaning |
| --- | --- |
| `Any of selected classes (OR)` | Show an image if it contains at least one selected class. |
| `All selected classes (AND)` | Show an image only if it contains every selected class. |

Count thresholds are optional. If you choose a maximum count for a class, the image must contain that class at least once and no more than the chosen number.

Example:

- Search class: `person`
- Max count: `2`
- Result: images with 1 or 2 detected people are shown, images with 3 or more people are hidden.

### 4. Results Display

The result grid opens each matching image with Pillow and draws detections:

- green boxes for classes included in the current search
- labels with class name and confidence score
- optional hiding of non-matching classes
- adjustable grid columns
- JSON export of the current results

## Configuration

Edit `configs/default.yaml` to change model behavior:

```yaml
model:
  yolo_model: "yolo11m.pt"
  conf_threshold: 0.3

data:
  image_extension: [".jpg", ".jpeg", ".png"]
```

Common changes:

- Increase `conf_threshold` to show fewer but more confident detections.
- Decrease `conf_threshold` to show more detections, including lower-confidence ones.
- Add more image extensions if your dataset uses other formats.

## Exporting Results

After a search:

1. Open `Export Options`.
2. Click `Download Results (JSON)`.
3. The downloaded JSON contains only the images that matched your current search.

This is useful if you want to pass matching image paths into another script or build a smaller dataset from the search results.

## Troubleshooting

### `ModuleNotFoundError`

Your virtual environment is probably not active, or dependencies were not installed.

```bash
pip install -r requirements.txt
```

### Streamlit command not found

Run Streamlit through Python:

```bash
python -m streamlit run app.py
```

### Metadata loads but images do not display

Metadata stores image paths. If the metadata was created on another computer or after moving folders, those paths may be invalid.

Fix it by either:

- regenerating metadata with `Process new images`, or
- editing `image_path` values in the metadata so they point to existing images.

The included sample metadata uses repo-relative paths so it works from the project root.

### No images are processed

Check that:

- the image folder path is correct
- images use `.jpg`, `.jpeg`, or `.png`
- the app is being run from the project root
- the folder is not empty

### CUDA is not being used

Run:

```bash
nvidia-smi
python -c "import torch; print(torch.cuda.is_available())"
```

If PyTorch prints `False`, install the CUDA build of PyTorch that matches your system.

### Model file not found

Make sure the model weights path is correct. The default project path is:

```text
yolo11m.pt
```

## Development Notes

Run the main app:

```bash
streamlit run app.py
```

Run the small Streamlit practice file:

```bash
streamlit run test/streamlit_basics.py
```

There is no full automated test suite yet. A good manual smoke test is:

1. Start the app.
2. Load `data/processed/coco-val-2017-500/metadata.json`.
3. Search for `apple`, `baseball bat`, and `bed`.
4. Confirm that 21 matching images are shown.
5. Toggle bounding boxes and export JSON.

## GitHub Push Checklist

Before pushing:

```bash
git status
git add README.md requirements.txt data/processed/coco-val-2017-500/metadata.json docs/screenshots/
git commit -m "Add professional README and screenshots"
git push
```

Recommended cleanup before publishing:

- Add a license file if you want others to reuse the project.
- Avoid committing virtual environments such as `.venv/`.
- Avoid committing generated cache folders such as `__pycache__/`.
- Keep large custom datasets outside Git unless they are small demo samples.

## License

No license file is currently included. Add a license before publishing if you want to define how other people can use, modify, or distribute this project.
