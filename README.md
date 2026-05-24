# VLI-25-06-Hoang

# CGAR: Confidence-Guided Adaptive Refinement for GUI Grounding

This repository provides the official implementation and evaluation code for **Confidence-Guided Adaptive Refinement (CGAR)**. This method is designed to improve GUI grounding performance specifically on high-resolution and complex-layout screens.

## 1. Prepare the Dataset for Benchmarking

Before running experiments, you need to download and organize the supported datasets.

1.  Navigate to the `dataset/` folder.
2.  Follow the specific instructions provided in that folder to download the data.
3.  **Important:** Ensure the directory structure matches exactly what the scripts expect.

**Supported Datasets:**
*   `screenspot`
*   `screenspot_v2`
*   `screenspot_pro`
*   `ui_vision`

---

## 2. Setup the Environment

We recommend using Conda to manage dependencies.

```bash
# Create and activate the environment
conda create -n cgar python=3.10
conda activate cgar

# Install dependencies
pip install -r requirements.txt

# Alternatively
conda create --name cgar --file requirements.txt
```


## 3. Run Experiments
### 3.1. Run CGAR Evaluation
After cloning this repository and preparing the dataset, you can evaluate the CGAR method using the evaluate_cgar.py script:

```bash
python evaluate_cgar.py \
    --dataset screenspot \
    --model_path microsoft/GUI-Actor-7B-Qwen2.5-VL \
    --data_root ./dataset \
    --output_file exps/cgar/results.jsonl \
    --max_crop_ratio 0.7 \
    --min_crop_ratio 0.3 \
    --max_iterations 6 \
    --shift_strategy shifting \
    --strategy_agg sum
```

#### Arguments

#### --dataset
Name of the dataset to evaluate on.

**Options:**
- screenspot
- screenspot_v2
- screenspot_pro
- ui_vision

---

#### --model_path
Model checkpoint or HuggingFace model name.

**Examples:**
- microsoft/GUI-Actor-7B-Qwen2.5-VL
- microsoft/GUI-Actor-3B-Qwen2.5-VL
- inclusionAI/V2P-7B

---

#### --max_crop_ratio
Maximum crop ratio for refinement.  
**Example:** 0.7

---

#### --min_crop_ratio
Minimum crop ratio for refinement.  
**Example:** 0.3

---

#### --max_iterations
Maximum number of iterations (including the initial prediction).
If you want to evaluate the performance after 3 refinement steps, max_iterations should be equal 4 at least.

**Example:**  
6 → 1 initial prediction + 5 refinement steps


---

#### --output_file
Path to save the evaluation results.

**Example:**  
```json
exps/cgar/GUI-Actor-7B-Qwen2.5-VL-ui_vision-shifting.jsonl
```
---
#### --shift_strategy
Boundary handling strategy for cropped regions.

**Options:**
- shifting
- clamping

---

#### --strategy_agg
Strategy to aggregate confidence scores across patches/tokens.

**Options:**
- sum
- max
- mean

---

#### --data_root
Root directory of the datasets.

**Example:**  
```json
./dataset
```
### 3.2. Compute Accuracy
After generating the result file using evaluate_cgar.py, run:

```bash
python compute_accuracy.py \
    --file exps/cgar/GUI-Actor-7B-Qwen2.5-screenspot_pro-max0.7_min0.3-10-shifting.jsonl \
    --conf 0.8 \
    --max_iter 3 \
    --samples 10
```


#### Arguments

#### --file
The CGAR result JSONL file produced in step 3.1.

**Example:**
```json
exps/cgar/GUI-Actor-7B-Qwen2.5-screenspot_pro-max0.7_min0.3-10-shifting.jsonl
```
---

#### --conf
- Confidence threshold used for stopping refinement early.
- Predictions with score >= conf stop refinement

**Example:** 0.75, 0.8


---
#### --max_iter
- Maximum number of iterations to consider when computing accuracy (may differ from the number used during evaluation).

**Example:** 3

---
#### --sample
- Number of rows per case in the result JSONL.
- This equals the number of iterations used during evaluation.

**Example:** 10

### 3.3. Hyperparameters

The following hyperparameters are those used in the paper:

- **For ScreenSpot and ScreenSpot-v2:** (Low-resolution configuration)  
- **For ScreenSpot-Pro and UI-Vision:** (High-resolution configuration)

| Hyper-parameter   | Low-resolution | High-resolution |
|-------------------|----------------|-----------------|
| `max_crop_ratio`  | 1.0            | 0.7             |
| `min_crop_ratio`  | 0.5            | 0.3             |
| `max_iter`  | 1              | 3               |
| `conf`  | 0.75              | 0.8               |





## 4. Output Format

The script writes results to `--output_file` in **JSON Lines (.jsonl)** format.  
Each line corresponds to **one iteration** of a sample.

---


### Fields Included

#### **id**
Index of the sample in the dataset.

---

#### **instruction**
Natural language instruction for the sample.

---

#### **bbox**
Ground-truth bounding box for the target region at the given iteration.

---

#### **pred**
Predicted location (coordinate).

---

#### **correct**
Whether the prediction lies inside the ground-truth bbox.

---

#### **score**
Confidence score produced at that iteration.

---

#### **repetive_number**
Iteration index:
- `1` = initial prediction (no refinement)
- `2, 3, ...` = refinement steps

---

### Summary

For each sample, the output file contains **max_iterations + 1 rows**:

- **1 row** → initial prediction  
- **Up to `max_iterations` rows** → refinement steps  
  (depending on confidence stopping conditions)

## 5. Visualize the results

You can use the Jupyter notebook below to inspect the attention maps and predictions over refinement iterations:
```json
visualize_attention_map.ipynb
```
This notebook allows you to:

- Load a sample from the dataset (e.g., ScreenSpot-Pro).
- Run the model to obtain:
  - the predicted point,
  - the confidence score,
  - the attention map at each iteration.
- Overlay:
  - the **ground-truth bounding box**,
  - the **predicted point**,
  - and the **attention heatmap** on the original image.



## Acknowledgments
We thank the authors of **V2P** and **GUI-Actor** for releasing their models and resources, 
which serve as strong baselines and greatly support our research.  
[[V2P](https://github.com/inclusionAI/AWorld-RL/tree/main/V2P)] | [[GUI-Actor](https://github.com/microsoft/GUI-Actor)]


## Reference

