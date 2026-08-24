# AOI 瑕疵影像分類

## 1. 專案說明

本專案使用 PyTorch 進行 AOI（Automatic Optical Inspection）瑕疵影像分類。

資料集共包含六種類別：

- 0：Normal
- 1：Void
- 2：Horizontal defect
- 3：Vertical defect
- 4：Edge defect
- 5：Particle

最終模型以 ImageNet 預訓練的 EfficientNet-B0 為基礎，並搭配 ColorJitter 資料增強與 Test-Time Augmentation（TTA）進行推論。

---

## 2. 執行環境

### 2.1 模型訓練環境

`train.py` 於 Google Colab 執行，並使用 GPU 進行模型訓練。

主要使用套件：

- Python 3
- PyTorch
- torchvision
- pandas
- NumPy
- scikit-learn
- Pillow
- matplotlib

`train.py` 會掛載 Google Drive，並從以下路徑讀取資料：

```
/content/drive/MyDrive/Colab Notebooks/final_project/
```

建議在 Google Colab 中開啟 GPU Runtime 後執行。

### 2.2 模型推論環境

`predict.py` 與 `predict_TTA.py` 可於本機 Python 環境執行。

主要使用套件：

- Python 3
- PyTorch 2.11.0 + CUDA 12.8
- torchvision 0.26.0 + CUDA 12.8
- pandas 3.0.5
- NumPy 2.4.6
- Pillow 12.3.0
- tqdm 4.70.0

本機使用支援 CUDA 的 PyTorch 版本進行 GPU 推論。
```

---

## 3. Training 資料準備

請先下載 AOI 資料集，並將 `aoi.zip` 放置於：


/content/drive/MyDrive/Colab Notebooks/final_project/
```

`aoi.zip` 內包含：

```
train.csv
test.csv
train_images.zip
test_images.zip
```

執行 `train.py` 後，程式會自動解壓縮資料集，並將原始訓練資料以 8:2 的比例切分為 training set 與 validation set。

---

## 4. 模型訓練


主要訓練設定如下：

- Model：EfficientNet-B0
- Pretrained weights：ImageNet
- Input size：224 × 224
- Batch size：32
- Loss function：Cross Entropy Loss
- Optimizer：AdamW
- Learning rate：1e-4
- Weight decay：1e-4
- Epochs：10
- Random seed：42

訓練階段使用以下資料增強：

- Random Horizontal Flip
- Random Vertical Flip
- ColorJitter
  - brightness = 0.1
  - contrast = 0.1

最佳模型選擇規則如下：

1. 優先選擇 Validation Accuracy 較高的模型。
2. 若 Validation Accuracy 相同，則選擇 Validation Loss 較低的模型。

訓練完成後，最佳模型權重會儲存為：

```text
best_efficientnet_b0.pth
```

---

## 5. Prediction 資料準備

進行本機推論時，請將檔案整理如下：

```text
project/
├── test.csv
├── best_efficientnet_b0.pth
├── predict.py
├── predict_TTA.py
└── test_images/
    └── test_images/
        └── *.png
```

`best_efficientnet_b0.pth` 為訓練完成後儲存的最佳模型權重。

---

## 6. 一般推論

執行：

```bash
python predict.py
```

程式會載入：

```text
best_efficientnet_b0.pth
```

並對 test images 進行分類。

輸出檔案為：

```text
prediction.csv
```

---

## 7. TTA 推論

執行：

```bash
python predict_TTA.py
```

TTA 對每張測試影像建立四種輸入：

1. Original
2. Horizontal Flip
3. Vertical Flip
4. Horizontal + Vertical Flip

四種影像分別輸入同一個 EfficientNet-B0 模型，最後將四組 logits 等權平均，並選擇平均後分數最高的類別作為最終預測結果。

目前 `predict_TTA.py` 會輸出：

```text
predict_TTA.csv
```

---

## 8. 最終方法

本專案最終採用：

```text
EfficientNet-B0
+ ImageNet pretrained weights
+ ColorJitter
+ Cross Entropy Loss
+ Equal-weight 4-view TTA
```

Public Leaderboard 最佳 Accuracy：

```text
0.9948212
```

最佳排名：

```text
69 / 1013
```
