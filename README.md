# TensorBoard OnDemand App

A BatchConnect app that runs TensorBoard inside a SLURM job and serves it securely through an authenticating reverse proxy. TensorBoard has no built-in auth, which is a problem on shared clusters. This app fixes that with a cookie-based gate so only your OnDemand session can reach the interface.

## How it works

When the job starts, two processes run:

1. `authrevproxy.py` starts on `$proxy_port` (the public-facing port). Every request is checked for a session cookie named `_ood_token_<session_id>`. The cookie value is compared against a token set in the job environment. If they match, the request is forwarded to TensorBoard. If not, it returns 403.

2. `tensorboard` starts on `$app_port` (internal only, bound to `127.0.0.1`). It never receives traffic directly.

The proxy is built with Twisted (`twisted.web.proxy.ReverseProxyResource`).

## Prerequisites

- OnDemand access on your cluster
- Python 3.10+ available via Lmod (or similar)
- A Python virtual environment with `twisted`, `tensorflow`, and `tensorboard` installed

## Setup

### Step 1: Load Python and create a virtual environment

```bash
module load lang/Python/3.10.8-GCCcore-12.2.0
python -m venv /path/to/your/tensorboard_env
source /path/to/your/tensorboard_env/bin/activate
```

### Step 2: Install dependencies

```bash
pip install twisted tensorflow tensorboard
```

### Step 3: Clone the app

```bash
git clone https://github.com/SammyCode002/TensorBoard ~/ondemand/dev/tensorboard
```

### Step 4: Update the script

Open `template/script.sh.erb` and replace the placeholder paths with your actual paths:

```bash
# Activate your virtual environment
source /path/to/your/tensorboard_env/bin/activate

# Path to authrevproxy.py
/path/to/your/tensorboard_env/bin/authrevproxy.py --app-port="$app_port" --proxy-port="$proxy_port" &

# Point TensorBoard at your actual log directory
tensorboard --logdir=/path/to/your/logs --host=127.0.0.1 --port="${app_port}"
```

## How the auth works

OnDemand sets a cookie `_ood_token_<session_id>` in your browser when you connect to a session. The same token is injected into the job's environment as `_ood_token_<session_id>`. The proxy reads both and requires them to match before forwarding any request. Anyone without the cookie gets a 403.

The session ID is derived from the job's working directory (`os.path.basename(os.getcwd())`), so each session has a unique token.

## Notes

- TensorBoard is started with `--logdir=/tmp` by default. Point it at your actual log directory.
- The proxy only runs while the batch job is alive. When the job ends, the session closes.
- Token auth covers access control but does not encrypt traffic. Use your cluster's VPN or OnDemand's HTTPS proxy for encryption.
