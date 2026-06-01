# P4DDLe: Introducing Packet-Level Analysis in Programmable Data Planes to Advance Network Intrusion Detection (Enhanced Version)

### 🚀 What's New in this Fork (Our Enhancements & Patches)
This repository is an optimized and enhanced version of the original P4DDLe framework. We have resolved critical architectural, deployment, and compilation bugs to make the system fully stable, multi-class capable, and compatible with modern containerized environments.

---

### 💻 Overcoming Hardware & Environment Challenges (The M1/M2 Mac Setup)
As developers and researchers, we often expect challenges to arise from algorithms or simulation logic, rather than baseline environment setup. However, the first major hurdle encountered in this project—even before writing a single line of code—was architectural incompatibility. Running TensorFlow and advanced networking tools directly on Apple Silicon (M1/M2 Mac) was impossible due to native compilation constraints built for Intel/Linux architectures.

💡 **The Engineering Solution: Containerization via Docker**
To bypass these hardware constraints and establish a rock-solid dev environment, a customized containerized workflow was engineered:

1. **Architecture Emulation:** A dedicated Docker container was deployed, forcing the emulation of the required target platform to provide a stable, fully compatible Ubuntu 22.04 environment:
   ```bash
   docker run --rm -it --privileged --platform linux/amd64 ubuntu:22.04 /bin/bash
   ```
2. **Environment Isolation:** A clean Python virtual environment (`venv`) was initialized inside the container to isolate dependencies and mitigate software library conflicts:
   ```bash
   python3 -m venv venv
   source ./venv/bin/activate
   ```
3. **Dependency Deployment:** Successfully compiled and installed `tensorflow==2.13.0` alongside advanced network emulation tools simultaneously without deployment friction:
   ```bash
   pip3 install tensorflow==2.13.0 scikit-learn h5py pyshark protobuf==3.19.6 mininet thrift psutil
   ```
4. **State Persistence:** Committed the final stable container state into a reusable image to guarantee instantaneous deployment in future sessions:
   ```bash
   docker commit <container_id> p4-dev
   ```

### 🐳 Automated Deployment via Dockerfile
While the initial environment troubleshooting was performed interactively, we have fully automated this entire deployment pipeline into a single, production-ready `Dockerfile` provided in this repository. 

To build and launch the environment on Apple Silicon without manual dependency installation, simply run:
```bash
# 1. Build the cross-platform container image
docker build --platform linux/amd64 -t p4-dev-image .

# 2. Launch the isolated development workspace
docker run --rm -it --privileged --platform linux/amd64 p4-dev-image /bin/bash
```
The system will boot instantly with Python 3.10, the virtual environment pre-configured, and TensorFlow successfully pinned to the environment path.

---

### 🛠️ Core Technical Enhancements Implemented (`lucid_cnn.py`)

#### 1. Docker Environment & M1/M2 Mac Compatibility Patch (Forced CPU Fallback)
* **Original Code (`lucid_cnn1.py`):** Tried to dynamically allocate GPU memory, which explicitly triggers system core dumps and allocation crashes inside emulated container layers on ARM64 architectures:
  ```python
  # tf.config.experimental.set_memory_growth(tf.config.list_physical_devices('GPU')[0], True)
  ```
* **Our Patch (`lucid_cnn.py`):** Explicitly bypassed host GPU visibility, forcing a stable, crash-free CPU execution flow:
  ```python
  tf.config.set_visible_devices([], 'GPU')
  ```

#### 2. Transition to Multi-Class Categorical Classification & Modern Keras Syntax
* **Original Code (`lucid_cnn1.py`):** Restricted to binary classification using a single-neuron output dense layer, binary cross-entropy loss, and deprecated `lr` parameter syntax:
  ```python
  model.add(Dense(1, activation='sigmoid', name='fc1'))
  optimizer = Adam(lr=lr, beta_1=0.9, beta_2=0.999, epsilon=None, decay=0.0, amsgrad=False)
  model.compile(loss='binary_crossentropy', optimizer=optimizer, metrics=['accuracy'])
  ```
* **Our Patch (`lucid_cnn.py`):** Upgraded the output layer architecture to support 2-class/categorical traffic tracking, adjusted the loss function, and updated the optimization syntax to comply with modern Keras standards:
  ```python
  model.add(Dense(2, activation='sigmoid', name='fc1'))
  optimizer = Adam(learning_rate=lr, beta_1=0.9, beta_2=0.999, epsilon=None, decay=0.0, amsgrad=False)
  model.compile(loss="categorical_crossentropy", optimizer="adam", metrics=["accuracy"])
  ```

#### 3. One-Hot Data Engineering & Adaptive Prediction Parsing (`np.argmax`)
* **Original Code (`lucid_cnn1.py`):** Parsed predictions statically based on a simple `0.5` binary threshold:
  ```python
  Y_pred_val = (best_model.predict(X_val) > 0.5)
  ```
* **Our Patch (`lucid_cnn.py`):** Embedded dynamic `to_categorical` parsing during compilation epochs. Integrated adaptive prediction parsing via `np.argmax` to extract peak class probability safely across varying tensor dimensions:
  ```python
  from tensorflow.keras.utils import to_categorical
  num_classes = len(set(Y_train))
  Y_train = to_categorical(Y_train, num_classes)
  Y_val = to_categorical(Y_val, num_classes)
  
  # Adaptive parsing execution block
  Y_pred_val = np.argmax(Y_pred_val, axis=1) if Y_pred_val.shape[1] > 1 else (Y_pred_val > 0.5).astype(int)
  ```

#### 4. Localization & Logging Standardization
Refactored the model's entire runtime logging and error-handling interface into clear English. Replaced native language constraints with standard terminal tracking for checkpoints (`Best model saved`), structural exception logging (`Error occurred while saving`), and ultimate state verification (`Model successfully saved`).

---

### 🔧 Compiler & Dependency Patches (External Submodules)

* **Compiler Build Automation Fix (`p4c` TC Backend):**  
  Patched `/app/p4c/backends/tc/CMakeLists.txt` by commenting out external repository fetching (`# fetchcontent_makeavailable(iproute2repo)`). This guarantees seamless, offline-compatible compilation using localized system libraries, preventing environment building from hanging.
  
* **Runtime Dependency Routing (`p4_util.py`):**  
  Fixed critical missing execution paths and runtime import exceptions by explicitly appending system directories for `bm_runtime` and `bmpy_utils` via `sys.path.append()`.

---

### Quick Start: Training & Testing the Model

Once inside your containerized environment, you can evaluate the optimized scripts using the following standard execution workflow:

#### 1. Multi-Class CNN Model Training:
To start the training process and export the validated network weights into your dataset path, run:
```bash
python3 lucid_cnn.py --train ./sample-dataset/ --epochs 100
```

#### 2. Attack Evaluation & Prediction:
To verify the intrusion detection accuracy against pre-processed evaluation logs, run:
```bash
python3 lucid_cnn.py --predict ./sample-dataset/ --model ./sample-dataset/latest_model.h5
```

> **Note:** The customized runtime paths embedded within `p4_util.py` will automatically intercept execution targets and route standard Python API objects smoothly to your local compiled behavioral model tools (`/app/behavioral-model/tools`).

---

More details on the architecture of P4DDLe, its performance and experiments are available in the following research paper:
*Doriguzzi-Corin, R., Knob, L. A. D., Mendozzi, L., Siracusa, D., and Savi, M. (2023). "Introducing packet-level analysis in programmable data planes to advance Network Intrusion Detection". *Computer Networks*, 110162.*
