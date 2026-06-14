# datasets-for-kurtosis

Two `.mat` files from the CWRU Bearing Dataset for the Machine Spotify Wrapped task.

---

## Files

| File | What it is |
|------|------------|
| `97_Normal_0.mat` | Healthy bearing, no fault |
| `OR007_6_1_136.mat` | Outer race fault, smallest severity |

---

## CWRU Naming Convention
```text
The file names aren't random — they encode everything about the experimental condition:

OR  007   6   1   136

│    │    │   │    │

│    │    │   │    └── file index (roughly maps to RPM ~1136)

│    │    │   └─────── load (1 HP)

│    │    └─────────── fault position (6 o'clock on the outer race)

│    └──────────────── fault size (0.007 inches — the smallest/mildest fault)

└───────────────────── fault type (OR = Outer Race)
```
Fault types across the full dataset: `OR` (outer race), `IR` (inner race), `B` (ball fault). Normal files are just labeled `Normal`.

---

## Loading the Files

Every `.mat` file is a dictionary. Always check the keys first:

```python
from scipy.io import loadmat

data = loadmat('OR007_6_1_136.mat')
print([k for k in data.keys() if not k.startswith('__')])
# ['X136_DE_time', 'X136_FE_time', 'X136RPM']
```

Use the `DE_time` key cuz that's the drive end accelerometer, which is the most commonly used channel for fault detection.

```python
signal = data['X136_DE_time'].flatten()[:4096]
```

The number in the key (`X136`) matches the file index in the filename. So `97_Normal_0.mat` → `X097_DE_time`, `OR007_6_1_136.mat` → `X136_DE_time`.

---

## Sampling Rate

All files here are recorded at `fs = 12000` Hz — 12,000 samples per second. Keep this consistent when computing FFT, otherwise your frequency axis will be wrong.

---

## Why only 4096 samples?

Each file contains ~120,000 samples. Loading all of them into an FFT and plotting with matplotlib is slow and unreadable. 4096 samples gives you roughly 0.34 seconds of signal which is more than enough to see the fault pattern clearly and compute meaningful statistics.
