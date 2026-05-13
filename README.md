# COVID-19 Detection and Localization

This repository contains my Optum internship project for the Kaggle
**SIIM-FISABIO-RSNA COVID-19 Detection** competition. The project uses chest
X-ray images to classify COVID-19 pneumonia appearance and localize lung
opacities with bounding boxes.

The competition asked participants to identify and localize COVID-19
abnormalities on chest radiographs. Each study was classified into one of four
study-level labels:

- Negative for Pneumonia
- Typical Appearance
- Indeterminate Appearance
- Atypical Appearance

The detection task focused on finding radiographic opacities in the image. The
challenge was hosted by SIIM, FISABIO, and RSNA on Kaggle, using augmented
annotations from public chest radiograph datasets including MIDRC-RICORD and
BIMCV-COVID-19.

## Project Overview

The repository is organized as a notebook-based workflow:

| Notebook | Purpose |
| --- | --- |
| `dcmtopng.ipynb` | Converts Kaggle DICOM chest X-ray files into PNG images using `pydicom`. |
| `label_creation.ipynb` | Merges image-level and study-level labels, prepares class labels, and creates YOLO-format text files for train and validation images. |
| `Object_detection.ipynb` | Trains a YOLOv5 object detection model to detect opacity regions in chest X-rays. |
| `Classification_final.ipynb` | Builds classification features from detection outputs and compares SVM, Decision Tree, KNN, Random Forest, and a small neural network. |
| `web_app.ipynb` | Creates a Flask + ngrok demo app for uploading an X-ray image and returning the predicted class. |

## Dataset

The Kaggle competition data contains chest radiographs in DICOM format with
image-level bounding box annotations and study-level class labels. In this
project, the notebooks convert DICOM files to PNG, map study labels to numeric
classes, and create object detection labels.

Class mapping used in the notebooks:

```text
0 -> Negative for Pneumonia
1 -> Typical Appearance
2 -> Indeterminate Appearance
3 -> Atypical Appearance
```

The classification notebook records 6,334 processed training images. The
label-creation notebook records:

- 4,334 training images with generated YOLO labels
- 1,367 validation images with generated YOLO labels

## Methodology

1. **DICOM preprocessing**
   - Read original DICOM files with `pydicom`.
   - Apply VOI LUT when available.
   - Correct monochrome inversion when needed.
   - Normalize images and export them as PNG files.

2. **Label preparation**
   - Load Kaggle image-level and study-level CSV files.
   - Remove Kaggle suffixes such as `_image` and `_study` from identifiers.
   - Join image metadata with study labels using `StudyInstanceUID`.
   - Convert bounding boxes to YOLO format.
   - Split labels into train and validation folders.

3. **Object detection**
   - Train YOLOv5s using image size `640`, batch size `16`, and `20` epochs.
   - Use the trained detector to identify opacity regions on test images.
   - Save predicted bounding boxes as YOLO text labels.

4. **Classification**
   - Use bounding box-derived features for class prediction.
   - Compare multiple classical machine learning models.
   - Train a small dense neural network as an additional baseline.

5. **Deployment demo**
   - Load the saved classification model.
   - Run YOLOv5 detection on uploaded images.
   - Convert detections into model input features.
   - Return the predicted COVID-19 appearance class through a Flask web page.

## Results

### Classification

Saved outputs in `Classification_final.ipynb` show the following validation/test
accuracies:

| Model | Accuracy |
| --- | ---: |
| SVM, RBF kernel | 73.80% |
| Decision Tree | 73.17% |
| K-Nearest Neighbors | 73.49% |
| Random Forest | 72.70% |
| Dense Neural Network | 73.86% |
| End-to-end test prediction pipeline | 52.76% |

The best standalone classifier result recorded in the notebook is the dense
neural network at approximately **73.86% accuracy**, closely followed by the SVM
at **73.80% accuracy**.

### Object Detection

Saved YOLOv5 training output in `Object_detection.ipynb` shows the final
validation metrics after 20 epochs:

| Metric | Value |
| --- | ---: |
| Precision | 0.798 |
| Recall | 0.233 |
| mAP@0.5 | 0.236 |
| mAP@0.5:0.95 | 0.071 |

The detection notebook also records inference on **633 test images**, with
predicted opacity labels saved under the YOLOv5 detection output folder.

## Requirements

The notebooks were developed in Google Colab and use Google Drive paths. Main
libraries include:

- Python
- NumPy
- pandas
- scikit-learn
- TensorFlow/Keras
- PyTorch
- YOLOv5
- OpenCV
- pydicom
- Flask
- flask-ngrok

Install the core Python dependencies with:

```bash
pip install numpy pandas scikit-learn tensorflow torch torchvision opencv-python pydicom flask flask-ngrok
```

YOLOv5 should be cloned or installed separately from the Ultralytics repository
before running the object detection notebook.

## How to Run

1. Download the SIIM-FISABIO-RSNA COVID-19 Detection data from Kaggle.
2. Upload or mount the data in Google Drive.
3. Run `dcmtopng.ipynb` to convert DICOM images to PNG.
4. Run `label_creation.ipynb` to generate classification labels and YOLO label
   files.
5. Run `Object_detection.ipynb` to train YOLOv5 and generate opacity detections.
6. Run `Classification_final.ipynb` to train and evaluate the classification
   models.
7. Run `web_app.ipynb` to launch the Flask demo.

The notebooks currently use paths under:

```text
/content/drive/MyDrive/Optum/
```

Update these paths if your dataset is stored somewhere else.

## Repository Structure

```text
COVID-19detection/
+-- Classification_final.ipynb
+-- Object_detection.ipynb
+-- dcmtopng.ipynb
+-- label_creation.ipynb
+-- web_app.ipynb
+-- README.md
```

## Notes

- This project is for learning and research purposes.
- The model is not intended for clinical diagnosis.
- Performance depends on the preprocessing paths, train/validation split, and
  saved YOLO/model artifacts used in Google Drive.
- Large datasets and trained model weights are not included in this repository.

## References

- Kaggle competition:
  <https://www.kaggle.com/c/siim-covid19-detection>
- SIIM challenge page:
  <https://siim.org/research-journal/siim-machine-learning-challenges/covid-19-kaggle-challenge/>
- RSNA challenge summary:
  <https://www.rsna.org/artificial-intelligence/ai-image-challenge/covid-19-al-detection-challenge-2021>
- Dataset annotation paper:
  <https://pmc.ncbi.nlm.nih.gov/articles/PMC9518934/>
