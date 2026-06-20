# How to use opencode with local llm using mlx in Apple device (Apple silicon)

1. Install opencode
    Personally I prefer to install this on venv.
    - Activate your venv and run in terminal
    ``` bash
    curl -fsSL https://opencode.ai/install | bash
    ```
2. Install mlx-lm (If you haven't installed it)
    Run 
    ``` bash
    pip install mlx-lm
    ```
    or
     Run 
    ``` bash
    conda install -c conda-forge mlx-lm
    ```   
3. modify opencode config

    Open `~/.config/opencode/opencode.json` and for example, pass the following (if you already have a config just add the MLX provider):

   ``` json
    {
        "$schema": "https://opencode.ai/config.json",
        "provider": {
            "mlx": {
            "npm": "@ai-sdk/openai-compatible",
            "name": "MLX (local)",
            "options": {
                "baseURL": "http://127.0.0.1:8080/v1"
            },
            "models": {
                "mlx-community/NVIDIA-Nemotron-3-Nano-30B-A3B-4bit": {
                    "name": "Nemotron 3 Nano"
                },
                "mlx-community/Qwen2.5-Coder-7B-Instruct-8bit": {
                    "name": "Qwen 2.5 Coder"
        }
            }
            }
        }
    }
   ```
4. deploy mlx-lm server
   
    ``` bash
    mlx_lm.server
    ```

5. Start OpenCode and select the provider
   
    In the repo you plan to work, type opencode.

    Once inside the OpenCode TUI:

    1. Enter `/connect`
    2. Type `MLX` and select it
    3. For the API key enter none
    4. Select the model
    5. Start planning and building
   

# Reference
- https://gist.github.com/awni/93a973a0cf5fb539b2ce1f37ec4a9989