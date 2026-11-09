CoinVision
A Python desktop application that detects coins in an image, classifies each one into a denomination (₹1, ₹2, ₹5, ₹10), and displays the per-denomination count and total value through a full-screen Tkinter GUI.
Files Required
File
Purpose
main.py
The entire application (GUI + preprocessing + detection + inference)
coin_classifier_finetuned3.h5
Pre-trained Keras model, loaded at startup
Both files must be in the same directory, because the model is loaded by a relative path.
Dependencies
cv2            # opencv-python
numpy          # numpy
tkinter        # standard library (on Linux may need: sudo apt install python3-tk)
tensorflow     # tensorflow.keras.models.load_model
PIL            # Pillow (Image, ImageTk)
os             # standard library (imported but not used)
Install with:
pip install opencv-python numpy tensorflow Pillow