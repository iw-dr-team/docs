## Hướng dẫn tích hợp Codex với LLMProxy

### Bước 1: Truy cập [link](https://portal.inwave.vn/llm-api/) để lấy API Key của bạn.
### Bước 2: Mở file `config.toml` của codex để setup. Path thường nằm ở ***"C:\Users\ducnv\.codex\config.toml"***
### Bước 3: Dán thông tin sau vào file `config.toml`

```toml
model = "cx/gpt-5.5"
model_provider = "local-proxy"
model_reasoning_effort = "high"
[model_providers.local-proxy]
name = "Local Proxy"
base_url = "https://llmproxy.inwave.vn/v1"
env_key = "OPENAI_API_KEY"
wire_api = "responses"
```

Lưu ý: Bạn có để đổi mode khác nhau tùy theo nhu cầu như `cx/gpt-5.5` hoặc `cx/gpt-5.6-sol`

### Bước 4: Set OPENAI_API_KEY với lệnh 
Windows:
```bash
setx OPENAI_API_KEY "YOUR_API_KEY"
```

MacOS/Linux:
```bash
echo 'export OPENAI_API_KEY="YOUR_API_KEY"' >> ~/.zshrc
source ~/.zshrc
```

### Bước 5: Khởi động lại Codex và test. Nhiều trường hợp bạn sẽ cần khởi động lại máy để biến môi trường được nhận diện.