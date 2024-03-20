# TensorBoard OnDemand App

This Tensorboard OnDemand app is secured by an authenticating reverse proxy. TensorBoard's web interface does not provide any inherent authentication, which is not ideal in a shared environment. Therefore, to enhance security, we use a browser cookie for authentication. The cookie is set on the OnDemand interactive app page and verified by the authenticating reverse proxy to grant access to the TensorBoard interface.

## Installation

To install and run this TensorBoard OnDemand app, follow the steps below:

### Step 1: Load the Python Module

First, load the Python module available on your system.

```bash
module load lang/Python/3.10.8-GCCcore-12.2.0

Step 2: Create a Virtual Environment
Create a virtual environment for TensorBoard.
python -m venv /path/to/tensorboard_env

Step 3: Activate the Virtual Environment
Activate the virtual environment.
source /path/to/tensorboard_env/bin/activate

Step 4: Install Dependencies
Install the required Python packages, including twisted, which is needed for the reverse proxy.
python -m pip install twisted
python -m pip install tensorflow
python -m pip install tensorboard

Step 5: Run the Authentication Reverse Proxy
Start the authrevproxy.py with the appropriate ports.
/path/to/authrevproxy.py --app-port="$app_port" --proxy-port="$proxy_port" &

Step 6: Start TensorBoard
Run TensorBoard, specifying the directory for log files.
tensorboard --logdir=/path/to/your/logs --host=127.0.0.1 --port="$app_port"

