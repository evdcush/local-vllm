#==================================  setup  ==================================#

# clone this repo:
git clone --depth=1 https://github.com/evdcush/local-vllm.git

# or just rip the files:
wget https://raw.githubusercontent.com/evdcush/local-vllm/refs/heads/master/docker-compose.yml;
wget https://raw.githubusercontent.com/evdcush/local-vllm/refs/heads/master/env-example;

##  DEPS
##  you're going to need docker, nvidia, nvidia docker and all that stuff isntalled
##  it's a common procedure, and only briefly mentioned here (end of file)

# setup the secrets
cp env-example .env


#=================================  send it  =================================#


# let 'er rip
docker compose up -d

# check on it
docker compose logs --no-log-prefix -f vllm

##  it will take some time to download the model and for vllm to get ready
##  you should see some stuff like:
##  docker compose logs --no-log-prefix -f vllm
##
##  INFO 04/20 04:20:30 [__init__.py:216] Automatically detected platform cuda.
##  (APIServer pid=1) INFO 04/20 04:20:32 [api_server.py:1839] vLLM API server version 0.11.0
##  (APIServer pid=1) INFO 04/20 04:20:32 [utils.py:233] non-default args: {'host': '0.0.0.0', 'model': 'nvidia/NVIDIA-Nemotron-Nano-9B-v2', 'trust_remote_code': True, 'gpu_memory_utilization': 0.85}
##  (APIServer pid=1) The argument `trust_remote_code` is to be used with Auto classes. It has no effect here and is ignored.
##  (APIServer pid=1) A new version of the following files was downloaded from https://huggingface.co/nvidia/NVIDIA-Nemotron-Nano-9B-v2:
##  <snipped>
##  INFO 04/20 04:20:46 [__init__.py:216] Automatically detected platform cuda.
##  (EngineCore_DP0 pid=178) INFO 04/20 04:20:48 [core.py:644] Waiting for init message from front-end.
##  (EngineCore_DP0 pid=178) INFO 04/20 04:20:48 [core.py:77] Initializing a V1 LLM engine (v0.11.0) with config: model='nvidia/NVIDIA-Nemotron-Nano-9B-v2', speculative_config=None, tokenizer='nvidia/NVIDIA-Nemotron-Nano-9B-v2', <SNIPPED>
##  (EngineCore_DP0 pid=178) W1102 04:20:49.154000 178 torch/utils/cpp_extension.py:2425] TORCH_CUDA_ARCH_LIST is not set, all archs for visible cards are included for compilation.
##  (EngineCore_DP0 pid=178) W1102 04:20:49.154000 178 torch/utils/cpp_extension.py:2425] If this is not desired, please set os.environ['TORCH_CUDA_ARCH_LIST'] to specific architectures.
##  <snipped>
##  (EngineCore_DP0 pid=178) INFO 04/20 04:23:22 [gpu_model_runner.py:3480] Graph capturing finished in 13 secs, took 0.58 GiB
##  (EngineCore_DP0 pid=178) INFO 04/20 04:23:22 [core.py:210] init engine (profile, create kv cache, warmup model) took 21.09 seconds
##  (APIServer pid=1) INFO 04/20 04:23:26 [loggers.py:147] Engine 000: vllm cache_config_info with initialization after num_gpu_blocks is: 286
##  (APIServer pid=1) INFO 04/20 04:23:26 [api_server.py:1634] Supported_tasks: ['generate']
##
##  <YOUJ NEED TO SEE THESE LINES BEFORE YOU CAN SEND IT!>
##
##  (APIServer pid=1) INFO 04/20 04:23:27 [api_server.py:1912] Starting vLLM API server 0 on http://0.0.0.0:8000
##  (APIServer pid=1) INFO:     Started server process [1]
##  (APIServer pid=1) INFO:     Waiting for application startup.
##  (APIServer pid=1) INFO:     Application startup complete

# expose to tialnet
tailscale serve reset
tailscale serve --bg --https=443 http://127.0.0.1:8000
tailscale serve status
## should see something like:
##   https://<your-device>.ts.net (tailnet only)
##   |-- / proxy http://127.0.0.1:8000


#---------------------------------  test it  ---------------------------------#


#===  test that sucker out  (NB: different model is used from the logs above)
## you can use your tailnet or simple localhost
# curl https://<your-device>.ts.net/v1/models | jq .
curl http://127.0.0.1:8000/v1/models | jq .
{
  "object": "list",
  "data": [
    {
      "id": "google/gemma-3-4b-it",
      "object": "model",
      "created": 1761530784,
      "owned_by": "vllm",
      "root": "google/gemma-3-4b-it",
      "parent": null,
      "max_model_len": 131072,
      "permission": [
        {
          "id": "modelperm-5ea9a4c358dfaszabd91dbcb1a9aff61",
          "object": "model_permission",
          "created": 1761530784,
          "allow_create_engine": false,
          "allow_sampling": true,
          "allow_logprobs": true,
          "allow_search_indices": false,
          "allow_view": true,
          "allow_fine_tuning": false,
          "organization": "*",
          "group": null,
          "is_blocking": false
        }
      ]
    }
  ]
}


# noyce 👌🏻


#===  test more (just some simple mic-check/sanity-test chat completions)
# curl -s https://<your-thign>.ts.net/v1/chat/completions \
curl -s http://127.0.0.1:8000/v1/chat/completions \
-H 'Content-Type: application/json' \
-d '{
      "model": "google/gemma-3-4b-it",
      "messages": [{"role":"user","content":"You must respond strictly in-character only using onomatopoeia and sounds in your response. Im driving my awesome, straightpiped mk3 supra (YOU) at high boost. A tunnel is coming up, no other cars. Im going to drop the hammer and just send it. As my beloved and dope mk3 supra, I love that sound of the 2JZ and the twin turbskis you make:"}]
    }' \
| jq .


{
  "id": "chatcmpl-6cc8f6dcd0fe432c9fc4965d6233ad40",
  "object": "chat.completion",
  "created": 1761531583,
  "model": "google/gemma-3-4b-it",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "*Vroom!* *Wooooosh!* *Rrrrrrr-RRRRRRRRR!* *Pshhhhhh!* \n\n*Thump-thump-thump!* (Engine revving) *Hrrrmmm!* (Turbos spooling) *Ffffffffffffft!* (Air rushing) \n\n*Screeeeeech!* (Tires gripping) *Vrrrooooom!* *Whoosh!* *Rumble-rumble!* (Tunnel approach) \n\n*BAM!* (Hammer dropped) *RRRRRRRRRRRRRRRR!* (Boost hitting) *Wooooosh!* *Shhhhhh!* (Sonic boom – almost!) \n\n*Purrrrrrrrrr* (Engine settling) *Shhhhhh.* (Speed) \n\n*Tick-tock.* (Clock - time flies!)",
        "refusal": null,
        "annotations": null,
        "audio": null,
        "function_call": null,
        "tool_calls": [],
        "reasoning_content": null
      },
      "logprobs": null,
      "finish_reason": "stop",
      "stop_reason": 106,
      "token_ids": null
    }
  ],
  "service_tier": null,
  "system_fingerprint": null,
  "usage": {
    "prompt_tokens": 94,
    "total_tokens": 285,
    "completion_tokens": 191,
    "prompt_tokens_details": null
  },
  "prompt_logprobs": null,
  "prompt_token_ids": null,
  "kv_transfer_params": null
}


#===  you're good to go


#===================================  deps  ==================================#


# REFER TO THE OFFICIAL DOCUMENTATION, NOT MY PERFUNCTORY DUMP HERE.
##
##    WARNING:
##    you are taking your life into your own hands
##    you are responsible for the outcomes in your lief, no one else
##    including me
##    expect to eat shit
##    tjat's how you learn and do stuff
##    so take this caca as a positive signal that you are learning
##    have confidence in yourself and take responsibiltiy for your shit
##
##    **do not assume any links or commands are up-to-date**
##    **or correct**
##    **or safe/secure**

#------------------------  nvidia drivers, CUDA, etc.  -----------------------#

## not fully covered here
## i've done this so many times i don't know what to refer source-wise
## here's my ref, from ym setup_notes:
## https://github.com/evdcush/Chest/blob/77594ebc9ed2fdba8b42012ce7cdb561ff9e9972/Setup/jammy_setup.sh#L114-L127

## know that youi need to use Xorg, not wayland garb

# First-things-first, CUDA. If CUDA is fine, everything else is manageable.
# (USING VERSIONS AT THE TIME OF INSTALLATION)

#=== Get the network installer deb.
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb

#=== (Remove any existing cuda keys that may have been installed by default buntu 3rd party setup):
sudo apt-key del 7fa2af80

#=== Install CUDA keys.
sudo dpkg -i cuda-keyring_1.0-1_all.deb

#=== Install CUDA.
#   Note: I'm not using pinned versions; I don't need to. If you do, just install that ver.
# NB: yous hould 10000000% target a version, eg cuda-12-9
#     don't be me.
sudo apt update && sudo apt install -y cuda

#=== CUDA SDK packages (after reboot!).
sudo apt install -y libcudnn8

# rprobly good to reboot again

#=== "confirm" you good:
nvidia-smi

## you should see something like:
# Fri April  20 04:20:00 1969
# +-----------------------------------------------------------------------------------------+
# | NVIDIA-SMI 575.57.08              Driver Version: 575.57.08      CUDA Version: 12.9     |
# |-----------------------------------------+------------------------+----------------------+
# | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
# | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
# |                                         |                        |               MIG M. |
# |=========================================+========================+======================|
# |   0  NVIDIA GeForce RTX 4090        On  |   00000000:01:00.0  On |                  N/A |
# .....


#-------------------------------  docker time  -------------------------------#


#=== add their key
sudo apt update;
sudo apt install ca-certificates curl;
sudo install -m 0755 -d /etc/apt/keyrings;
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc;
sudo chmod a+r /etc/apt/keyrings/docker.asc;

#=== add their apt shit to your apt sources
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null;

#=== install it
sudo apt update;
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#=== linux post-install quality of life stuff
# so you don't need to run sudo and shit evry time
sudo groupadd docker;
sudo usermod -aG docker $USER;

# logout or reboot (sane)

#=== activat echanges to groups live (maybe?)
newgrp docker

#=== verify:
docker run hello-world


#-------------------------  nvidia-container-toolkit  ------------------------#


## refer:
## https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

#===  get deps
sudo apt update && sudo apt install -y --no-install-recommends \
curl \
gnupg2

#===  get the key and add their bullshit to your apt sources
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && \
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list;

sudo apt update;

#===  install the nvidia container toolkit
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.18.0-1
sudo apt install -y \
nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION};


#=============================================================================#

that's all.
good luck.
