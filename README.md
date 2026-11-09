# CoinVision

A Python desktop application that detects coins in an image, classifies each one into a denomination (₹1, ₹2, ₹5, ₹10), and displays the per-denomination count and total value through a full-screen Tkinter GUI.

---

## Files Required

| File | Purpose |
| :--- | :--- |
| `main.py` | The entire application (GUI + preprocessing + detection + inference) |
| `coin_classifier_finetuned3.h5` | Pre-trained Keras model, loaded at startup |

Both files must be in the **same directory**, because the model is loaded by a relative path.

---

## Dependencies

```
cv2            # opencv-python
numpy          # numpy
tkinter        # standard library (on Linux may need: sudo apt install python3-tk)
tensorflow     # tensorflow.keras.models.load_model
PIL            # Pillow (Image, ImageTk)
os             # standard library (imported but not used)
```

Install with:

```bash
pip install opencv-python numpy tensorflow Pillow
```

---

## Running

```bash
python main.py
```

On launch:

- The window opens full-screen (`root.attributes('-fullscreen', True)`)
- The window is non-resizable (`root.resizable(False, False)`)
- The title is set to "COINVISION"
- The initial display shows:

```
Coin Count:
1 Rupee: 0
2 Rupees: 0
5 Rupees: 0
10 Rupees: 0

Total Value: 0 Rupees
```

There is no quit button in the current code. Close the app with Alt+F4 or your OS window manager.

---

## Usage

1. Click **Upload Image**
2. A file dialog opens, filtered to `*.png;*.jpg;*.jpeg`
3. Select an image
4. The processed image appears with green circles and blue denomination labels drawn on each detected coin
5. The count per denomination and the total value update below

If the dialog is cancelled (`file_path` is empty), the function returns without doing anything.

---

## How It Works

### 1. Startup

- Loads `coin_classifier_finetuned3.h5` via `load_model()`
- Class index to label mapping:
  ```python
  {0: "1 Rupee", 1: "10 Rupees", 2: "2 Rupees", 3: "5 Rupees"}
  ```
- Label to value mapping:
  ```python
  {"1 Rupee": 1, "10 Rupees": 10, "2 Rupees": 2, "5 Rupees": 5}
  ```
- Builds the Tkinter layout: top frame (upload button), image frame (image panel), result frame (count label + total label)

### 2. Image Loading

- `cv2.imread(file_path)`
- Resized to 1280 × 960 using `cv2.INTER_LINEAR`

### 3. Preprocessing

| Step | Call | Parameters |
| :--- | :--- | :--- |
| Grayscale | `cv2.cvtColor` | `COLOR_BGR2GRAY` |
| Gaussian Blur | `cv2.GaussianBlur` | kernel (7, 7), sigma 1.5 |
| Adaptive Threshold | `cv2.adaptiveThreshold` | `ADAPTIVE_THRESH_GAUSSIAN_C`, `THRESH_BINARY_INV`, blockSize 15, C 2 |
| Morphological Close | `cv2.morphologyEx` | `MORPH_CLOSE`, 3 × 3 ones kernel |

> **Note:** the threshold and morphology steps produce a variable `thresh` that is not used afterwards. Coin detection runs on `blurred`, not on `thresh`.

### 4. Coin Detection

```python
circles = cv2.HoughCircles(
    blurred,
    cv2.HOUGH_GRADIENT,
    dp=1.2,
    minDist=60,
    param1=50,
    param2=30,
    minRadius=30,
    maxRadius=100
)
```

- If `circles` is `None`, the detection block is skipped and the image is displayed unchanged with counts at 0
- Otherwise circles are rounded to `uint16`, sorted by radius descending, and sliced to the top 4 (`[:4]`)

### 5. Per-Coin Processing

For each of the (up to) 4 circles:

| Step | Operation |
| :--- | :--- |
| Radius expand | `adjusted_radius = int(r * 1.2)` |
| Bounding box | Clamped to image bounds with `max(0, …)` and `min(shape, …)` |
| Crop | `image[y1:y2, x1:x2]` |
| Resize | `cv2.resize(roi, (224, 224))` |
| Colour convert | `cv2.cvtColor(…, COLOR_BGR2RGB)` |
| Normalize | `roi_rgb / 255.0` |
| Batch dimension | `np.expand_dims(…, axis=0)` |
| Predict | `model.predict(roi_expanded)` |
| Class | `np.argmax(predictions)`, looked up in `class_labels` |
| Accumulate | Adds to `coin_counts[label]` and `total_value` |
| Annotate | Green circle `(0, 255, 0)`, thickness 2, at `(x, y)` with radius `adjusted_radius` |
| Label | Blue text `(255, 0, 0)`, `FONT_HERSHEY_SIMPLEX`, scale 0.8, thickness 2, at `(x-40, y-10)` |

### 6. Display

- Annotated image converted BGR to RGB
- Converted to `PIL.Image`
- Resized to 800 × 600 using `Image.Resampling.LANCZOS`
- Converted to `ImageTk.PhotoImage` and set on the panel label
- Reference retained on `panel.image` to prevent garbage collection
- `count_label` updated with the newline-joined `coin_counts`
- `result_label` updated with `f"Total Value: {total_value} Rupees"`

---

## Behaviour Notes

| Observation | Detail |
| :--- | :--- |
| Max 4 coins | `[:4]` slice applied after sorting by radius |
| Detection input | `blurred` is passed to `HoughCircles`; the thresholded image is not used |
| `os` imported but unused | No `os.*` call appears in the code |
| `thresh` computed but unused | Threshold + morphology output is not passed to any subsequent function |
| Count label order shifts after processing | Initial static text orders as 1, 2, 5, 10; the `coin_counts` dict iterates as 1, 10, 2, 5 |
| No error handling on `cv2.imread` | If the file fails to load (returns `None`), the subsequent resize call will raise |
| No quit path in GUI | The only way to exit is via the OS window manager |
| Window | Fixed `resizable(False, False)` combined with fullscreen |
| Static images only | Input is via `filedialog.askopenfilename`; no webcam or video code present |
| Fonts | Upload button Arial 14; count label Arial 14; total label Arial 16 bold |

---

## What Is Not in the Code

The following are not present in `main.py`:

- Any training code
- Any dataset loading or splitting
- Any confusion matrix or accuracy calculation
- Any augmentation
- Any model definition (only `load_model`)
- Any video / webcam / live capture
- Any currency other than the four mapped classes
- Any error handling around model loading or image reading
- Any unit tests

---

## License

No license file is included with this project.
