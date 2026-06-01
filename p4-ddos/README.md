## P4DDLe (Enhanced & Multi-Class Capable)

### Introduction

We use an altered, enhanced version of LUCID as the P4ddle/Baseline control-plane. The main differences present in this optimized version are complete architectural stability inside Docker on modern hardware (ARM64/M1/M2), built-in multi-class categorical classification capabilities, and fully standard English terminal logging.

---

### 🐳 Important Deployment Note for Docker Users
If you are deploying this project within our pre-configured Docker container environment (optimized for Mac M1/M2/ARM64 and Linux via cross-compilation), **you can completely skip Step 1, Step 2, Step 3, and Step 4**. 

All dependencies, virtual environments, standard network emulators (`mininet`, `tcpreplay`), and behavioral model source layers come pre-compiled and fully provisioned out-of-the-box. Simply launch your container and proceed directly to **Step 5**.

---

### Prerequisites (For Native Linux Environments Only)

- Ubuntu Linux 22.04
- Git
- Python 3.10
- Pip3

### Step 1: Installing Required Packages

First, let's ensure we have the necessary packages installed:

```bash
sudo apt update
sudo apt install -y python3-pip python3-venv
```

### Step 2: Setting Up Python Virtual Environment

Next, let's create and activate a Python virtual environment:

```bash
python3 -m venv venv
source ./venv/bin/activate
# to exit the virtual environment run the command: deactivate
```

### Step 3: Installing Dependencies and Mininet

Now, let's install the required Python dependencies and mininet:

```bash
sudo apt install -y tshark mininet tcpreplay
pip3 install tensorflow==2.13.0 scikit-learn h5py pyshark protobuf==3.19.6 mininet thrift psutil
```

### Step 4: Installing and Compiling P4 Compiler and Behavioral Model

We'll begin by installing the P4 Compiler from its repository:

```bash
source /etc/lsb-release
echo "deb http://download.opensuse.org/repositories/home:/p4lang/xUbuntu_\${DISTRIB_RELEASE}/ /" | sudo tee /etc/apt/sources.list.d/home:p4lang.list
curl -fsSL "https://download.opensuse.org/repositories/home:p4lang/xUbuntu_\${DISTRIB_RELEASE}/Release.key" | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/home_p4lang.gpg > /dev/null
sudo apt update
sudo apt install p4lang-p4c
```

> ⚠️ **Compilation Patch Update:** The `p4c` (TC backend) package compiler pipeline has been patched in `/app/p4c/backends/tc/CMakeLists.txt` by commenting out external repository fetching (`# fetchcontent_makeavailable(iproute2repo)`). This bypasses network-bound stalls and forces compilation via pre-installed localized system libraries.

Unfortunately, the BMv2 package does not contain the python libraries needed to run P4ddle, so we need to manually compile it (remember that you need to have the virtual environment activated):

```bash
git clone https://github.com/p4lang/behavioral-model.git
cd behavioral-model
./autogen.sh 
./configure --with-python_prefix=\$VIRTUAL_ENV
make -j3
sudo make install
```

### Step 5: Preparing the Dataset

Before we train our model, we need to prepare our dataset:

```bash
cd p4-ddos
python3 control-plane/lucid_dataset_parser.py --dataset_type DOS2019 --dataset_folder sample-dataset/ --packets_per_flow 10 --dataset_id DOS2019 --traffic_type all --time_window 4 --p4_compatible
python3 control-plane/lucid_dataset_parser.py --preprocess_folder sample-dataset/
```

### Step 6: Training the Deep Learning Model

Now, to train the optimized multi-class/categorical model, let's run:

```bash
python3 control-plane/lucid_cnn.py --train sample-dataset/
```
*Note: Our custom script automatically handles Keras dynamic `to_categorical` label expanding and outputs execution traces fully in standard English terminology.*

### Step 7: Performing Live Predictions

After training the model, we can perform multi-class predictions on both sample data and live traffic using the exported latest network weights:

```bash
# test set evaluation using the newly standardized model output name
python3 control-plane/lucid_cnn.py --predict sample-dataset/ --model sample-dataset/latest_model.h5

# predict a given pcap using the modern categorical evaluation workflow
python3 control-plane/lucid_cnn.py --predict_live sample-dataset/CIC-DDoS-2019-DNS.pcap --model sample-dataset/latest_model.h5 --dataset_type DOS2019
```

### Step 8: Initiating P4 Switch Implementation

Now, we need to start the P4 Switch. To do that, we need to compile and start it in a separate terminal (again venv mandatory):

```bash
# Open a new terminal
cd data-plane/baseline
p4c --target bmv2 --arch v1model --std p4-16 p4_packet_management.p4

cd ../runner
sudo python3 launcher.py --behavioral-exe simple_switch --json ../baseline/p4_packet_management.json --cli simple_switch_CLI
```
*Note: Runtime routing paths inside `control-plane/p4_util.py` are already updated via `sys.path.append()` to guarantee seamless hookups with the local runtime standard tools (`/app/behavioral-model/tools`).*

### Step 9: Generating Traffic

While the P4 switch is running, let's generate traffic for testing (activate venv!):

```bash
# Open a new terminal
sudo python3 helpers/traffic_generator.py -f sample-dataset/CIC-DDoS-2019-DNS.pcap -i s1-eth2 -d 600
```

### Step 10: Live Predictions with the P4 Switch

Finally, with the switch running and with the traffic generator active, let's perform live categorical predictions using our optimized LUCID script:

```bash
# In the first terminal
python3 control-plane/lucid_cnn.py --predict_live localhost:22222 --model sample-dataset/latest_model.h5 --dataset_type DOS2019 -r 14 -si baseline
```

### Conclusion

After some time, you can close the 3 terminals. All the evaluation telemetry data generated by LUCID will be safely structured and exported to your localized log folder.

To run the full p4ddle implementation instead of the baseline setup, simply switch directories on step 8, and on step 10 replace the option `--si baseline` with `--si p4ddle`.
