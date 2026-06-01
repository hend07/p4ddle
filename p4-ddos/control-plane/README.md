## Control Plane Setup & Experimentation

### Parameter Configuration
In the `experiment_**.sh` script, a few execution parameters must be configured before running the emulation suite:
* The target PCAP files containing the attack data and benign traffic traces.
* The explicit flow volume and benign number of packets per second.
* The specific packet sampling rates (up to 1 in 16; higher rates require structural modification within the data plane P4 files).
* The definitive test duration for each cross-combination of speed, sampling, and register sizing.
* The exact name of your trained multi-class network weights (e.g., `latest_model.h5`).

### Execution Permissions
To allow the automated evaluation script to orchestrate traffic replay seamlessly without root privilege boundaries, update the executable permissions for `tcpreplay`:
```bash
sudo chmod a+s /usr/bin/tcpreplay
```

### 🧠 Modernized Dependency & Routing Layer (No Conda Required)
Unlike the legacy documentation which restricted deployment to outdated Anaconda blocks (`python38`), our optimized repository standardizes execution on **Python 3.10 via native virtual environments (`venv`)**. 

* **Automated Package Resolution:** Critical interaction modules such as `psutil` (for automated benchmarking orchestration) and `thrift` (for runtime switch controller communication) are already fully bundled and injected during our core environment setup stage.
* **Automated API Routing Fix:** The historical constraint requiring manual code modification on line 2 of `p4_util.py` to prevent interpreter crashes has been **completely eliminated**. Our update embeds resilient directory mapping directly into the file execution header via dynamic `sys.path` appending, mapping external standard components seamlessly.

### Running the Experiments
Once the shell configuration matches your target testing scenario, trigger the evaluation pipeline directly within your active environment:
```bash
./experiment_**.sh
```
