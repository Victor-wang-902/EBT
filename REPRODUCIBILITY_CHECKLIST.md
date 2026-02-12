# Reproducibility Checklist

Use this checklist to reproduce the main training and inference workflows in this repository.

## 1) Environment and Dependencies

- [ ] Clone the repository and enter it:
  - `git clone https://github.com/alexiglad/EBT.git`
  - `cd EBT`
- [ ] Create and activate the recommended conda environment:
  - `conda create -n ebt python=3.12`
  - `conda activate ebt`
- [ ] Install dependencies:
  - `pip install -r requirements.txt`
- [ ] If needed, use an alternative dependency set:
  - `pip install -r gh200_requirements.txt` (GH200 setup)
  - `pip install -r loose_requirements.txt` (without pinned GPU stack)
- [ ] (Optional) Create environment from YAML:
  - `conda env create -f environment.yml`
- [ ] Configure Hugging Face cache/token as needed:
  - `export HF_HOME=/your/cache/path`
  - `export HF_TOKEN=...`
- [ ] Login to Weights & Biases:
  - `wandb login`

## 2) Data Setup

- [ ] Prepare datasets for your target modality.
- [ ] For video experiments, follow the data setup in `data/vid/README.md`.
- [ ] Ensure all dataset paths in your selected job script point to valid local/shared storage.

## 3) Baseline Sanity Check (Fast)

- [ ] Run the minimal NLP example to verify your environment:
  - `python example_code/minimal_nlp_training_loop.py`
- [ ] Confirm training logs run without import/runtime errors.

## 4) Choose a Reproduction Target

- [ ] Pick a modality and model type from `job_scripts/`:
  - NLP pretrain: `job_scripts/nlp/pretrain/`
  - NLP inference: `job_scripts/nlp/inference/`
  - Image pretrain/inference: `job_scripts/img/...`
  - Video pretrain/inference: `job_scripts/vid/...`
- [ ] Pick EBT or baseline Transformer script.
- [ ] Record the exact script path and commit hash for your run log.

## 5) Configure the Job Script

- [ ] Open your selected script and update run metadata consistently:
  - `RUN_NAME`
  - `MODEL_NAME`
  - `MODEL_SIZE`
- [ ] Set your W&B `entity` and `project`.
- [ ] Verify dataset arguments and output/checkpoint directories.
- [ ] For inference scripts, set:
  - `--only_test_model_ckpt` to a real checkpoint file
  - `--only_test`
  - `--execution_mode "inference"` when appropriate

## 6) Launch Training/Inference

- [ ] Run directly (quick path):
  - `bash <your_job_script.sh>`
- [ ] Or run through slurm wrapper (HPC path):
  - `bash slurm_executor.sh reference_a100 <your_job_script.sh>`
- [ ] For multinode (if needed), follow `job_scripts/nlp/pretrain/ebt_s1_mn.sh` guidance (`ntasks = ngpus`, `srun python ...`).

## 7) Track and Validate Outputs

- [ ] Save stdout/stderr logs for each run.
- [ ] Save exact CLI arguments or job script snapshot used.
- [ ] Confirm checkpoints are written to expected output directory.
- [ ] Confirm W&B metrics/curves are visible and complete.
- [ ] For inference runs, collect generated outputs and evaluation metrics.

## 8) Repeatability Controls

- [ ] Run each key experiment at least 2-3 times (if resources allow).
- [ ] Keep the following fixed across repeats unless intentionally changing them:
  - Code commit
  - Job script
  - Hardware allocation
  - Dataset version/location
- [ ] Document any deviations from default scripts.

## 9) Reporting

- [ ] Produce a run table with:
  - Script path
  - Commit hash
  - Hardware (GPU type/count)
  - Total steps/epochs
  - Final metrics
  - Checkpoint path
- [ ] Compare EBT vs baseline runs using matched settings whenever possible.
- [ ] Link all artifacts (logs, checkpoints, W&B run URLs) in your final report.

## 10) Optional Debugging Utilities

- [ ] Use `job_scripts/debug/` scripts for debugging dataloaders and code paths.
- [ ] Use `utils/dataloader_debugger.py` and `utils/find_corrupt_files.py` when diagnosing data issues.

---

## Suggested Minimal Reproduction Path

If you want a straightforward start, follow this exact order:

1. Set up environment (`requirements.txt` + `wandb login`).
2. Run `python example_code/minimal_nlp_training_loop.py`.
3. Run one NLP baseline pretrain script (`job_scripts/nlp/pretrain/baseline_transformer.sh`).
4. Run one NLP EBT pretrain script (`job_scripts/nlp/pretrain/ebt_s1.sh`).
5. Run matching inference scripts in `job_scripts/nlp/inference/` using produced checkpoints.
6. Compare metrics and qualitative generations.
