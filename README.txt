# setup
##  clone this repo:
git clone --depth=1 https://github.com/evdcush/local-vllm.git

## or just rip the files:
wget https://raw.githubusercontent.com/evdcush/local-vllm/refs/heads/master/docker-compose.yml

wget https://raw.githubusercontent.com/evdcush/local-vllm/refs/heads/master/env-example


# regarding the deps and such
# you're going to need docker, nvidia, nvidia docker and all that stuff
# installed


# setup the secrets
cp env-example .env

# let 'er rip
docker compose up -d

# check on it
docker compose logs --no-log-prefix -f vllm

docker ps
docker logs --tail=200 vllm-vllm-1
# or excessive:
docker logs --tail=200 $(docker ps --filter "name=vllm" --format "{{.Names}}")


# expose to tialnet
tailscale serve reset
tailscale serve --bg --https=443 http://127.0.0.1:8000
tailscale serve status


# test that sucker out
## you can use your tailnet or simple localhost
$ curl http://127.0.0.1:8000/v1/models | jq .
$ curl https://<your-device>.ts.net/v1/models | jq .
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




noyce 👌🏻


# test more (just some simple mic-check/sanity-test chat completions)
#$ curl -s http://127.0.0.1:8000/v1/chat/completions \
$ curl -s https://<your-thign>.ts.net/v1/chat/completions \
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
