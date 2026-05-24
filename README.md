# Secondary Structure Predictor for 3AVE

This project predicts the secondary structure of the human IgG1 Fc fragment protein structure **3AVE** using a custom Support Vector Machine (SVM) model.

The model is trained on external PDB structures and then evaluated on the test protein **3AVE** by comparing the predicted secondary structure labels against DSSP-derived ground truth labels.

## Project Objective

The goal of this project is to predict the three-state secondary structure of the 3AVE protein chain:

- `H` = Helix
- `E` = Beta strand
- `C` = Coil

The final performance is measured using the **Q3 score**, which represents the percentage of residues whose predicted secondary structure matches the true DSSP-assigned secondary structure.

## Protein Context

**PDB ID:** 3AVE  
**Protein:** Human IgG1 Fc fragment  
**Chain used:** A  

The 3AVE structure represents an immunoglobulin-like Fc fragment, which is expected to contain a high proportion of beta-strand secondary structure due to its beta-sandwich fold.

## Project Workflow

The project has two main scripts:

### 1. `trained_svm.py`

This script trains an SVM model for secondary structure prediction.

It performs the following steps:

1. Loads external PDB files from a training folder.
2. Extracts chain A residues from each PDB file.
3. Uses DSSP to obtain experimental secondary structure labels.
4. Converts DSSP 8-state labels into Q3 labels:
   - Helix-like: `H`, `G`, `I` → `H`
   - Beta-like: `E`, `B` → `E`
   - Coil-like: `T`, `S`, `-` → `C`
5. Encodes each amino acid using a sliding-window approach.
6. Trains a Support Vector Machine classifier.
7. Saves the trained model as `svm_q3_model.pkl`.

### 2. `predict_3ave.py`

This script uses the trained SVM model to predict the secondary structure of 3AVE.

It performs the following steps:

1. Loads the trained model file `svm_q3_model.pkl`.
2. Extracts the primary amino acid sequence from `3ave.pdb`.
3. Encodes the sequence using the same sliding-window method used during training.
4. Predicts Q3 labels for each residue.
5. Extracts the true DSSP Q3 labels from `3ave.pdb`.
6. Calculates the Q3 score for the trained SVM model.
7. Compares the result against a JPred baseline prediction.

## Repository Structure

```text
secondary-structure-predictor/
│
├── trained_svm.py
├── predict_3ave.py
├── 3ave.fasta
├── 3ave.pdb
├── requirements.txt
├── .gitignore
└── README.md
```

## Important Note About the Trained Model

The trained model file:

```text
svm_q3_model.pkl
```

is required to run `predict_3ave.py`, but it is not included in this repository because of its file size.

To generate it locally, run:

```bash
python trained_svm.py
```

After training, the model will be saved automatically as:

```text
svm_q3_model.pkl
```

Then you can run the prediction script.

## Training Data Note

The training script expects a local folder of PDB files.

Inside `trained_svm.py`, the training folder is defined as:

```python
PDB_FOLDER = "/home/lenovo555/protein_project/pdbs"
```

If you run this project on another machine, update this path to the folder that contains your training PDB files.

Example:

```python
PDB_FOLDER = "/path/to/your/pdb/files"
```

The script processes chain `A` from each PDB file and skips files that cannot be processed by DSSP or do not meet the sequence length criteria.

## Requirements

Install the required Python packages using:

```bash
pip install -r requirements.txt
```

The main Python dependencies are:

```text
biopython
joblib
numpy
scikit-learn
```

This project also requires **DSSP** to be installed and accessible from the system, because Biopython uses DSSP to assign secondary structure labels from PDB files.

## How to Run the Project

### 1. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 2. Install requirements

```bash
pip install -r requirements.txt
```

### 3. Train the SVM model

Before running prediction, train the model:

```bash
python trained_svm.py
```

Example training output:

```text
✔ Successfully processed PDB files: 83
✔ Skipped PDB files: 17
✔ Total windows: 17148
✔ Feature vector size: 147
✔ Trained model saved as svm_q3_model.pkl
SVM Accuracy (Sliding Window): 81.49%
```

### 4. Predict secondary structure for 3AVE

After generating `svm_q3_model.pkl`, run:

```bash
python predict_3ave.py
```

Example prediction output:

```text
Primary sequence of 3AVE:
LLGGPSVFLFPPKPKDTLMISRTPEVTCVVVDVSHEDPEVKFNWYVDGVEVHNAKTKPREEQYNSTYRVVSVLTVLHQDWLNGKEYKCKVSNKALPAPIEKTISKAKGQPREPQVYTLPPSRDELTKNQVSLTCLVKGFYPSDIAVEWESNGQPENNYKTTPPVLDSDGSFFLYSKLTVDKSRWQQGNVFSCSVMHEALHNHYTQKSLSLS

Sequence length: 211

✔ Q3 score of trained SVM on 3AVE: 72.51%

✔ Q3 score of JPred on 3AVE: 71.09%
```

## Model Method

The model uses a **sliding-window encoding** approach.

For each residue, the model looks at a window of neighboring amino acids:

```text
Window size = 7
```

Each amino acid is represented using one-hot encoding. The model then predicts the secondary structure label of the center residue.

The final feature vector size is:

```text
147
```

This comes from:

```text
7 positions × 21 amino acid symbols
```

where the 21 symbols include the 20 standard amino acids plus the padding symbol `X`.

## DSSP to Q3 Mapping

DSSP originally provides 8-state secondary structure labels. In this project, they are converted into 3-state Q3 labels:

| DSSP Label | Q3 Label | Meaning |
|---|---|---|
| `H`, `G`, `I` | `H` | Helix-like |
| `E`, `B` | `E` | Beta-like |
| `T`, `S`, `-` | `C` | Coil-like |

## Results

| Model / Method | Q3 Score |
|---|---:|
| Custom SVM model | 72.51% |
| JPred baseline | 71.09% |

The custom SVM model achieved a slightly higher Q3 score than the JPred baseline on the 3AVE test case.

## Files Required to Run Prediction

To run `predict_3ave.py`, the following files must be present in the project root directory:

```text
predict_3ave.py
3ave.pdb
svm_q3_model.pkl
```

`3ave.pdb` is included in the repository.  
`svm_q3_model.pkl` must be generated locally by running `trained_svm.py`.

## Limitations

This is an educational machine learning project for protein secondary structure prediction.

Current limitations include:

- The model uses a simple sliding-window encoding.
- It does not use evolutionary profiles such as PSSM or HMM features.
- It does not use deep learning or protein language model embeddings.
- The model performance depends heavily on the quality and diversity of the training PDB dataset.
- The current evaluation focuses on one test structure: 3AVE.
- The training dataset path is currently configured locally and should be updated before running on another machine.

## Future Improvements

Possible improvements include:

- Use a larger and more diverse training dataset.
- Add PSSM-based evolutionary features.
- Add cross-validation across PDB structures.
- Evaluate on multiple independent test proteins.
- Try Random Forest, Gradient Boosting, or neural network models.
- Compare against additional public secondary structure prediction tools.
- Organize input/output folders for larger-scale experiments.

## Technologies Used

- Python
- Biopython
- DSSP
- NumPy
- Scikit-learn
- Joblib
- Support Vector Machine (SVM)