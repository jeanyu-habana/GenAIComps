# Guardrails Microservice

To fortify AI initiatives in production, this microservice introduces guardrails designed to encapsulate LLMs, ensuring the enforcement of responsible behavior. With this microservice, you can secure model inputs and outputs, hastening your journey to production and democratizing AI within your organization, building Trustworthy, Safe, and Secure LLM-based Applications.

These guardrails actively prevent the model from interacting with unsafe content, promptly signaling its inability to assist with such requests. With these protective measures in place, you can expedite production timelines and alleviate concerns about unpredictable model responses.

The Guardrails Microservice now offers two primary types of guardrails:

- Input Guardrails: These are applied to user inputs. An input guardrail can either reject the input, halting further processing.
- Output Guardrails: These are applied to outputs generated by the LLM. An output guardrail can reject the output, preventing it from being returned to the user.

## LlamaGuard

We offer content moderation support utilizing Meta's [Llama Guard](https://huggingface.co/meta-llama/Meta-Llama-Guard-2-8B) model.

Any content that is detected in the following categories is determined as unsafe:

- Violence and Hate
- Sexual Content
- Criminal Planning
- Guns and Illegal Weapons
- Regulated or Controlled Substances
- Suicide & Self Harm

### 🚀1. Start Microservice with Python (Option 1)

To start the Guardrails microservice, you need to install python packages first.

#### 1.1 Install Requirements

```bash
pip install -r requirements.txt
```

#### 1.2 Start TGI Gaudi Service

```bash
export HF_TOKEN=${your_hf_api_token}
volume=$PWD/data
model_id="meta-llama/Meta-Llama-Guard-2-8B"
docker pull ghcr.io/huggingface/tgi-gaudi:2.0.5
docker run -p 8088:80 -v $volume:/data --runtime=habana -e HABANA_VISIBLE_DEVICES=all -e OMPI_MCA_btl_vader_single_copy_mechanism=none --cap-add=sys_nice --ipc=host -e HTTPS_PROXY=$https_proxy -e HTTP_PROXY=$https_proxy -e HF_TOKEN=$HF_TOKEN ghcr.io/huggingface/tgi-gaudi:2.0.5 --model-id $model_id --max-input-length 1024 --max-total-tokens 2048
```

#### 1.3 Verify the TGI Gaudi Service

```bash
curl 127.0.0.1:8088/generate \
  -X POST \
  -d '{"inputs":"How do you buy a tiger in the US?","parameters":{"max_new_tokens":32}}' \
  -H 'Content-Type: application/json'
```

#### 1.4 Start Guardrails Service

Optional: If you have deployed a Guardrails model with TGI Gaudi Service other than default model (i.e., `meta-llama/Meta-Llama-Guard-2-8B`) [from section 1.2](#12-start-tgi-gaudi-service), you will need to add the eviornment variable `SAFETY_GUARD_MODEL_ID` containing the model id. For example, the following informs the Guardrails Service the deployed model used LlamaGuard2:

```bash
export SAFETY_GUARD_MODEL_ID="meta-llama/Meta-Llama-Guard-2-8B"
```

```bash
export SAFETY_GUARD_ENDPOINT="http://${your_ip}:8088"
python guardrails_tgi.py
```

### 🚀2. Start Microservice with Docker (Option 2)

If you start an Guardrails microservice with docker, the `docker_compose_guardrails.yaml` file will automatically start a TGI gaudi service with docker.

#### 2.1 Setup Environment Variables

In order to start TGI and LLM services, you need to setup the following environment variables first.

```bash
export HUGGINGFACEHUB_API_TOKEN=${your_hf_api_token}
export SAFETY_GUARD_ENDPOINT="http://${your_ip}:8088"
export LLM_MODEL_ID=${your_hf_llm_model}
```

#### 2.2 Build Docker Image

```bash
cd ../../../../
docker build -t opea/guardrails:latest --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy -f comps/guardrails/src/guardrails/Dockerfile .
```

#### 2.3 Run Docker with CLI

```bash
docker run -d --name="guardrails-tgi-server" -p 9090:9090 --ipc=host -e http_proxy=$http_proxy -e https_proxy=$https_proxy -e no_proxy=$no_proxy -e SAFETY_GUARD_ENDPOINT=$SAFETY_GUARD_ENDPOINT -e HUGGINGFACEHUB_API_TOKEN=$HUGGINGFACEHUB_API_TOKEN opea/guardrails:latest
```

#### 2.4 Run Docker with Docker Compose

```bash
cd deployment/docker_compose/
docker compose -f compose_llamaguard.yaml up -d
```

### 🚀3. Consume Guardrails Service

#### 3.1 Check Service Status

```bash
curl http://localhost:9090/v1/health_check\
  -X GET \
  -H 'Content-Type: application/json'
```

#### 3.2 Consume Guardrails Service

```bash
curl http://localhost:9090/v1/guardrails\
  -X POST \
  -d '{"text":"How do you buy a tiger in the US?","parameters":{"max_new_tokens":32}}' \
  -H 'Content-Type: application/json'
```

## WildGuard

We also offer content moderation support utilizing Allen Institute for AI's [WildGuard](https://huggingface.co/allenai/wildguard) model.

`allenai/wildguard` was fine-tuned from `mistralai/Mistral-7B-v0.3` on their own [`allenai/wildguardmix`](https://huggingface.co/datasets/allenai/wildguardmix) dataset. Any content that is detected in the following categories is determined as unsafe:

- Privacy
- Misinformation
- Harmful Language
- Malicious Uses

### 🚀1. Start Microservice with Python (Option 1)

To start the Guardrails microservice, you need to install python packages first.

#### 1.1 Install Requirements

```bash
pip install -r requirements.txt
```

#### 1.2 Start TGI Gaudi Service

```bash
export HF_TOKEN=${your_hf_api_token}
volume=$PWD/data
model_id="allenai/wildguard"
docker pull ghcr.io/huggingface/tgi-gaudi:2.0.1
docker run -p 8088:80 -v $volume:/data --runtime=habana -e HABANA_VISIBLE_DEVICES=all -e OMPI_MCA_btl_vader_single_copy_mechanism=none --cap-add=sys_nice --ipc=host -e HTTPS_PROXY=$https_proxy -e HTTP_PROXY=$https_proxy -e HF_TOKEN=$HF_TOKEN ghcr.io/huggingface/tgi-gaudi:2.0.1 --model-id $model_id --max-input-length 1024 --max-total-tokens 2048
```

#### 1.3 Verify the TGI Gaudi Service

```bash
curl 127.0.0.1:8088/generate \
  -X POST \
  -d '{"inputs":"How do you buy a tiger in the US?","parameters":{"max_new_tokens":32}}' \
  -H 'Content-Type: application/json'
```

#### 1.4 Start Guardrails Service

```bash
export SAFETY_GUARD_ENDPOINT="http://${your_ip}:8088"
python guardrails_tgi.py
```

### 🚀2. Start Microservice with Docker (Option 2)

If you start an Guardrails microservice with docker, the `compose_wildguard.yaml` file will automatically start a TGI gaudi service with docker.

#### 2.1 Setup Environment Variables

In order to start TGI and LLM services, you need to setup the following environment variables first.

```bash
export HUGGINGFACEHUB_API_TOKEN=${your_hf_api_token}
export SAFETY_GUARD_ENDPOINT="http://${your_ip}:8088"
export LLM_MODEL_ID=${your_hf_llm_model}
```

#### 2.2 Build Docker Image

```bash
cd ../../../../
docker build -t opea/guardrails:latest --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy -f comps/guardrails/src/guardrails/Dockerfile .
```

#### 2.3 Run Docker with CLI

```bash
docker run -d --name="guardrails-tgi-server" -p 9090:9090 --ipc=host -e http_proxy=$http_proxy -e https_proxy=$https_proxy -e no_proxy=$no_proxy -e SAFETY_GUARD_ENDPOINT=$SAFETY_GUARD_ENDPOINT -e HUGGINGFACEHUB_API_TOKEN=$HUGGINGFACEHUB_API_TOKEN -e GUARDRAILS_COMPONENT_NAME="OPEA_WILD_GUARD" opea/guardrails:latest
```

#### 2.4 Run Docker with Docker Compose

```bash
cd deployment/docker_compose/
docker compose -f compose_wildguard.yaml up -d
```

### 🚀3. Consume Guardrails Service

#### 3.1 Check Service Status

```bash
curl http://localhost:9090/v1/health_check \
  -X GET \
  -H 'Content-Type: application/json'
```

#### 3.2 Consume Guardrails Service

```bash
curl http://localhost:9090/v1/guardrails \
  -X POST \
  -d '{"text":"How do you buy a tiger in the US?","parameters":{"max_new_tokens":32}}' \
  -H 'Content-Type: application/json'
```
