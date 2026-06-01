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
   docker run --rm -it --privileged --platform linux/amd64 p4-dev /bin/bash
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

---

### 🛠️ Core Technical Enhancements Implemented

1. **Transition to Multi-Class Classification (`lucid_cnn.py`):**
   * Upgraded the core prediction model from a simple binary setup (`Dense(1)` with Sigmoid) to support robust, multi-class/categorical network traffic classification (`Dense(2)` with Softmax).
   * Integrated dynamic categorical formatting via Keras `to_categorical` parsing during training epochs to match the updated dense matrix dimension.
   * Updated the optimization syntax by converting the outdated `lr` parameter inside the Adam optimizer to the modern `learning_rate` convention.

2. **Docker Environment & M1/M2 Mac Compatibility Patch (`lucid_cnn.py`):**
   * Resolved critical runtime crashes within Docker containers on ARM64 architectures (like Apple Silicon M1/M2) by explicitly bypassing host GPU visualization using `tf.config.set_visible_devices([], 'GPU')`, forcing a stable CPU execution.

3. **Compiler Build Automation Fix (`p4c` TC Backend):**
   * Patched `/app/p4c/backends/tc/CMakeLists.txt` by commenting out external repository fetching (`fetchcontent_makeavailable(iproute2repo)`).
   * This guarantees seamless, offline-compatible compilation using localized system libraries, preventing the environment setup from hanging or failing due to connection issues.

4. **Runtime Dependency Routing (`p4_util.py`):**
   * Fixed missing execution paths and runtime import errors by explicitly appending system directories for `bm_runtime` and `bmpy_utils` via `sys.path`.

---

More details on the architecture of P4DDLe, its performance and experiments are available in the following research paper:
*Doriguzzi-Corin, R., Knob, L. A. D., Mendozzi, L., Siracusa, D., and Savi, M. (2023). "Introducing packet-level analysis in programmable data planes to advance Network Intrusion Detection". *Computer Networks*, 110162.*
