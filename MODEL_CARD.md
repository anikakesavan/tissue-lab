# Tissue Lab model card

This is a new educational tissue classifier trained for the interactive demo, not an exported version of the original notebook's CNN or ResNet.

## Model and preprocessing

Frozen ImageNet-pretrained MobileNetV3 Small feature encoder, with a trained 576 → 128 → 8 classification head (Hardswish, dropout 0.3). Images are resized to 224 × 224 with bilinear interpolation. RGB inputs are float32 in [0,255]. Division by 255 and ImageNet channel normalization are embedded in the ONNX model. Class order: adipose, complex, debris, empty, lympho, mucosa, stroma, tumor.

## Data and evaluation

- 1,024 images from the original notebook's Inspirit AI dataset URL.
- 0 exact duplicate images removed before splitting.
- Stratified fixed split: 716 training / 154 validation / 154 test; seed 20260930.
- Horizontal flip, vertical flip, and 90-degree rotation are applied to training images only, alongside original views.
- Fixed encoder features, AdamW learning rate 0.002 and weight decay 0.01, label smoothing 0.05.
- Validation cross entropy selected epoch 16 from 36 training epochs. Early stopping patience: 20. Test results did not influence model selection.
- Test accuracy: 143/154 = 92.8571%.
- Tumor: precision 21/23 = 91.3043%; recall 21/21 = 100%; F1 95.4545%.
- Eleven test images were misclassified, including two false tumor predictions.

This is a same-source, image-level holdout. Patient identifiers are not available, so patient-level separation cannot be established. There is no external or clinical validation. These numbers do not estimate accuracy on arbitrary photographs, clinical slides, other datasets, or patient diagnoses. Scores are uncalibrated softmax outputs.

## Demo examples

For each class, the first correctly classified held-out image is selected when available. A known misclassified held-out example is also included. This curated gallery is not the evaluation set: aggregate metrics include all 154 reserved test images. New browser predictions execute ONNX inference and are never substituted with saved scores or labels.

## Verification

ONNX checker passed. ONNX Runtime CPU scores match the Python model on all nine gallery images within 0.000002 absolute difference. Chrome WebAssembly inference matches all nine labels and rounded displayed scores. Uploading the tumor sample reproduces the gallery result and marks its ground truth as unknown.

ONNX SHA-256: `b256887043e2d04c12fb7a08e85537521556ea1173ec16f20f2b5f99ac93374c`

Full metrics: `public/model/metrics.json`. Training notebook: `training/Tissue_Lab_Training.ipynb`. Split indices: `training/split.json`. Native export verification: `training/onnx_validation.json`.

## Intended use

Student learning and research demonstrations with histology tissue patches. Not for diagnosis, clinical decisions, or screening. The classifier always chooses among its eight classes and does not reliably detect out-of-distribution inputs.
