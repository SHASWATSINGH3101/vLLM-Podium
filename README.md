
# vLLM-kaggle 🚀

A self-contained setup for hosting a vLLM inference server on Kaggle, exposing it via ngrok, and connecting to it locally for experimentation.

<img src='https://github.com/aki-008/vLLM-Podium/blob/main/image/9d1c6b7c-eba8-43b4-8b39-d14af793cef9.jpeg' alt="vllm-podium" width="350">


## 📁 Repository Structure

```
aki-008-vllm-podium/
├── README.md    
└── src/
    ├── inference-server-vllm.ipynb   # Kaggle notebook to launch the vLLM server
    └── local_model_runner.ipynb      # Notebook to connect locally and query the server
```

## ✅ Prerequisites

- ⚙️ Kaggle environment with GPU support  
- 🐍 Python 3.8+  
- 🔧 vLLM and PyTorch installed on Kaggle  
- 🌐 `pyngrok` for creating public tunnels  
- 🔐 ngrok account (for auth token)  
- 💻 Local machine with Python + requests, LangChain, or similar libraries

## 🔧 1. Launching vLLM on Kaggle

1. Open `inference-server-vllm.ipynb` in your Kaggle workspace.  
2. Install dependencies:
   ```bash
   !pip install -U vllm torch
   ```
3. Configure and start the server:
   ```python
   import subprocess

   cmd = [
     "vllm", "serve", "Menlo/Jan-nano-128k",
     "--host", "0.0.0.0", "--port", "1234",
     "--enable-auto-tool-choice",
     "--tool-call-parser", "hermes",
     "--rope-scaling", '{"rope_type":"yarn","factor":3.2,"original_max_position_embeddings":40960}',
     "--max-model-len", "32768",
     "--tensor-parallel-size", "2",
     "--gpu-memory-utilization", "0.8"
   ]
   process = subprocess.Popen(cmd)
   print(f"Started vLLM with PID {process.pid}")
   ```
4. Verify the server is running:
   ```bash
   !ps -ax | grep vllm
   !netstat -tulnp | grep 1234
   ```

## 🌍 2. Exposing the Server via ngrok

1. Install `pyngrok`:
   ```bash
   !pip install pyngrok --quiet
   ```
2. Start an ngrok tunnel:
   ```python
   from pyngrok import ngrok

   ngrok.set_auth_token("YOUR_NGROK_TOKEN")
   tunnel = ngrok.connect(1234)
   print("Public URL:", tunnel.public_url)
   ```
3. Test the health endpoint:
   ```python
   import requests
   response = requests.get(f"{tunnel.public_url}/health")
   print("Server health:", response.status_code)
   ```
4. To close the tunnel:
   ```bash
   !killall ngrok
   ```

## 🔗 3. Connecting from Your Local Machine

1. Open `local_model_runner.ipynb`.  
2. Update the `base_url` to your ngrok URL:
   ```python
   BASE_URL = "https://<YOUR_NGROK_ID>.ngrok-free.app/v1"
   HEADERS = {"Authorization": "Bearer <API_KEY>"}
   ```
3. List available models:
   ```python
   import requests
   resp = requests.get(f"{BASE_URL}/models", headers=HEADERS)
   print(resp.json())
   ```
4. Use LangChain:
   ```python
   from langchain_openai import ChatOpenAI
   from langchain.schema import SystemMessage, HumanMessage

   llm = ChatOpenAI(
       model="Qwen/Qwen3-4B",
       openai_api_base=BASE_URL,
       openai_api_key="none",
       temperature=0.7,
   )

   messages = [
     SystemMessage(content="You are a helpful assistant."),
     HumanMessage(content="Hello! Summarize this article.")
   ]
   print(llm.invoke(messages).content)
   ```

## 🧹 4. Cleanup

- 🛑 Stop vLLM:
  ```bash
  !kill <vLLM_PID>
  ```
- ❌ Teardown ngrok:
  ```bash
  !killall ngrok
  ```

## 🛠️ Tips & Troubleshooting

- 🧠 **Model Names**: Use `/v1/models` to verify loaded models  
- 🔐 **Security**: Don’t expose your instance publicly without access control  
- 📊 **Resource Tuning**: Adjust `--tensor-parallel-size` and `--gpu-memory-utilization` as needed  
- 🪵 **Logs**: Log to file (`jan128k.log`) to debug errors

---

## 📌 Future Enhancements
- ✍️ More advanced post structuring and style customization
- 🔁 Enhanced user feedback loop for refining generated content
- 🌐 Integration with additional social media platforms (e.g., Twitter, Facebook)

---

## 🤝 Contributing
We welcome contributions! Fork the repo, create a new branch, and submit a pull request.

---

## 🪪 License
This project is licensed under the **MIT License**.

---

## 📚 Citing the Project

To cite this repository in publications:

```bibtex
@misc{CSVision,
  author = {aki-008},
  title = {vLLM-Podium},
  year = {2025},
  howpublished = {\url{https://github.com/aki-008/vLLM-Podium}},
  note = {GitHub repository},
}
```
