## ⚙️ Installation

**1. Clone the repository and create the conda environment:**

```bash
git clone https://github.com/IdolLab/RSTTrack.git
cd RSTTrack
conda create -n RSTTrack python=3.10
conda activate RSTTrack
```

**2. Download auxiliary models:**

- CLIP: [Baidu Drive](https://pan.baidu.com/s/1NREUlp9FJsK-FZV-SDDqgw?pwd=tea6) (pwd: `tea6`) / [Google Drive](https://drive.google.com/file/d/1fHGIc4NqrdLuE1D-pUqu7yPoI7duWPAq/view?usp=sharing)
- Qwen-VL: [Baidu Drive](https://pan.baidu.com/s/1H2ZSTpCElm9B0WswTDkzRA?pwd=61h4) (pwd: `61h4`) / [Google Drive](https://drive.google.com/file/d/1jloqePa0PLh38rSqX6tkTBSAI7phn2g0/view?usp=sharing)

**3. Install dependencies:**

```bash
pip install -r requirements.txt
```

---

## 📁 Data Preparation

Download the following datasets and place them under `./data/`:

- **RGB-T images**: [GTOT, RGBT210, RGBT234, LasHeR](https://chenglongli.cn/Datasets-and-benchmark-code/)
- **Textual annotations**: [Baidu Drive](https://pan.baidu.com/s/1Z_v8rrLdJt06hgYmcJjubQ?pwd=nayy) (pwd: `nayy`) / [Google Drive](https://drive.google.com/file/d/18GOOF6YKekhIwodJMI0ZQPwkRJHFuahI/view?usp=drive_link)

The expected directory structure is as follows:

```text
RSTTrack/
└── data/
    ├── GTOT/
    │   ├── BlackCar/
    │   │   ├── i/
    │   │   ├── v/
    │   │   ├── groundTruth_i.txt
    │   │   ├── groundTruth_v.txt
    │   │   ├── visible_description.txt
    │   │   └── class.txt
    │   └── ...
    ├── RGBT210/
    │   ├── afterrain/
    │   │   ├── infrared/
    │   │   ├── visible/
    │   │   ├── init.txt
    │   │   ├── visible_description.txt
    │   │   └── class.txt
    │   └── ...
    ├── RGBT234/
    │   └── ...  (same structure as RGBT210)
    └── LasHeR/
        ├── train/
        │   ├── 1boygo/
        │   │   ├── infrared/
        │   │   ├── visible/
        │   │   ├── init.txt
        │   │   └── visible_description.txt
        │   └── ...
        └── test/
            ├── 1blackteacher/
            │   ├── infrared/
            │   ├── visible/
            │   ├── init.txt
            │   ├── visible_description.txt
            │   └── class.txt
            └── ...
```

---

## 🔧 Setup & Configuration

Run the following command to initialize local paths:

```bash
python tracking/create_default_local_file.py --workspace_dir . --data_dir ./data --save_dir ./output
```

Alternatively, manually edit the path configs:

```text
./lib/train/admin/local.py      # paths for training
./lib/test/evaluation/local.py  # paths for testing
```

---

This repo is based on [DUTrack](https://github.com/GXNU-ZhongLab/DUTrack) and RAGTrack. We sincerely thank the authors for their excellent works.

---

## 

```bibtex


```

