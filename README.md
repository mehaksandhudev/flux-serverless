# Flux-Serverless: Autoscale Serverless Image Generation API



Deploy FLUX.1-dev and FLUX.1-schnell image generation models on [Modal](https://modal.com) with GPU acceleration using Nunchaku quantization for faster inference.

![Generated Sample](https://raw.githubusercontent.com/your-username/Flux-Free/main/test_image.png)

## ✨ Features

- **FLUX.1-dev**: High-quality 50-step generation with guidance
- **FLUX.1-schnell**: Fast 4-step generation
- **Nunchaku Quantization**: 4-bit quantized transformers for efficient GPU usage
- **REST API**: Simple HTTP POST endpoint for integration
- **Serverless**: Auto-scaling with Modal, pay only when running

---

## 🚀 Quick Start

### Prerequisites

- Python 3.10+
- [Modal account](https://modal.com) (free tier available)
- [Hugging Face account](https://huggingface.co) with access to FLUX models

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/Flux-Free.git
cd Flux-Free
```

### 2. Install Dependencies

```bash
# Create virtual environment
python -m venv .venv

# Activate (Windows)
.\.venv\Scripts\activate

# Activate (Linux/Mac)
source .venv/bin/activate

# Install Modal
pip install modal
```

### 3. Sign Up for Modal

1. Go to [modal.com](https://modal.com) and sign up
2. Install and authenticate the Modal CLI:
   ```bash
   python -m modal setup
   ```
3. Follow the browser prompt to authenticate

### 4. Get Hugging Face Access Token

1. Go to [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens)
2. Create a new access token with **Read** permissions
3. **Important**: Accept the FLUX model licenses:
   - [FLUX.1-dev](https://huggingface.co/black-forest-labs/FLUX.1-dev) - Click "Agree and access repository"
   - [FLUX.1-schnell](https://huggingface.co/black-forest-labs/FLUX.1-schnell) - Click "Agree and access repository"

### 5. Create Modal Secret

Create a secret named `flux-app-secrets` in Modal with your tokens:

**Option A: Via Modal Dashboard**

1. Go to [modal.com/secrets](https://modal.com/secrets)
2. Click **"Create new secret"**
3. Name it: `flux-app-secrets`
4. Add the following environment variables:

| Key | Value | Description |
|-----|-------|-------------|
| `BEARER_TOKEN` | `your-api-key` | Any string to protect your API (e.g., `mySecretKey123`) |
| `HF_TOKEN` | `hf_xxxxx...` | Your Hugging Face access token |

![Modal Secrets Screenshot](docs/modal_secrets.png)

**Option B: Via Command Line**

```bash
python -m modal secret create flux-app-secrets \
  BEARER_TOKEN="your-api-key" \
  HF_TOKEN="hf_your_huggingface_token"
```

### 6. Deploy to Modal

**Development mode (hot reload):**
```bash
python -m modal serve modal_nunchaku_flux_dev.py
```

**Production deployment:**
```bash
python -m modal deploy modal_nunchaku_flux_dev.py
```

After deployment, you'll see your API URL:
```
✓ Created web function fastapi_app => https://YOUR-USERNAME--nunchaku-flux-dev-fastapi-fastapi-app.modal.run
```

---

## 📡 API Reference

### Endpoint

```
POST /generate
```

### Headers

| Header | Value | Required |
|--------|-------|----------|
| `Content-Type` | `application/json` | Yes |
| `Authorization` | `Bearer YOUR_BEARER_TOKEN` | Yes (if BEARER_TOKEN is set) |

### Request Body

```json
{
  "prompt": "a beautiful sunset over mountains",
  "width": 1024,
  "height": 1024,
  "steps": 50,
  "guidance_scale": 3.5,
  "seed": 12345
}
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prompt` | string | *required* | Text description of the image (max 280 characters) |
| `width` | integer | 1024 | Image width in pixels (must be divisible by 8, min 256) |
| `height` | integer | 1024 | Image height in pixels (must be divisible by 8, min 256) |
| `steps` | integer | 50 | Number of denoising steps (higher = better quality, slower) |
| `guidance_scale` | float | 3.5 | How closely to follow the prompt (higher = more literal) |
| `seed` | integer | *random* | Seed for reproducible results |

### Constraints

- **Maximum resolution**: 1 megapixel (1,048,576 pixels total)
  - Valid: 1024×1024, 2048×512, 768×1024
  - Invalid: 2048×1024 (exceeds 1MP)
- **Minimum resolution**: 256×256
- **Dimensions**: Must be divisible by 8

### Response

- **Success (200)**: Returns PNG image as binary data
- **Error (400)**: Invalid parameters
- **Error (401)**: Invalid or missing bearer token
- **Error (500)**: Server error

---

## 🔧 Usage Examples

### cURL

```bash
curl -X POST "https://YOUR-URL/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_BEARER_TOKEN" \
  -d '{"prompt": "a cute cat sitting on a rainbow", "width": 512, "height": 512, "steps": 20}' \
  -o output.png
```

### Python

```python
import requests

url = "https://YOUR-URL/generate"
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer YOUR_BEARER_TOKEN"
}
payload = {
    "prompt": "a futuristic cityscape at night",
    "width": 1024,
    "height": 1024,
    "steps": 50,
    "guidance_scale": 3.5
}

response = requests.post(url, json=payload, headers=headers)

if response.status_code == 200:
    with open("generated_image.png", "wb") as f:
        f.write(response.content)
    print("Image saved!")
else:
    print(f"Error: {response.text}")
```

### JavaScript (Node.js)

```javascript
const fetch = require('node-fetch');
const fs = require('fs');

async function generateImage() {
  const response = await fetch('https://YOUR-URL/generate', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer YOUR_BEARER_TOKEN'
    },
    body: JSON.stringify({
      prompt: 'a magical forest with glowing mushrooms',
      width: 1024,
      height: 1024,
      steps: 50
    })
  });

  if (response.ok) {
    const buffer = await response.buffer();
    fs.writeFileSync('output.png', buffer);
    console.log('Image saved!');
  } else {
    console.error('Error:', await response.text());
  }
}

generateImage();
```

---

## 🔗 n8n Integration

Use the **HTTP Request** node to integrate with your n8n workflows:

### Node Configuration

| Setting | Value |
|---------|-------|
| **Method** | `POST` |
| **URL** | `https://YOUR-URL/generate` |
| **Authentication** | `Generic Credential Type` → `Header Auth` |
| **Name** | `Authorization` |
| **Value** | `Bearer YOUR_BEARER_TOKEN` |
| **Send Body** | `ON` |
| **Body Content Type** | `JSON` |
| **Specify Body** | `Using JSON` |

### Body (JSON)

```json
{
  "prompt": "{{ $json.prompt }}",
  "width": 512,
  "height": 512,
  "steps": 20,
  "guidance_scale": 3.5
}
```

### Important Settings

- **Timeout**: Set to `120000` ms (120 seconds) for cold starts
- **Response Format**: `File` (to receive the PNG image)

### Example Workflow

1. **Trigger** → Webhook or Schedule
2. **HTTP Request** → Call FLUX API with prompt
3. **Move Binary Data** → Process the image
4. **Write to File** / **Upload to S3** / **Send via Email**

---

## 📁 Project Structure

```
Flux-Free/
├── modal_nunchaku_flux_dev.py     # FLUX.1-dev deployment (50 steps, high quality)
├── modal_nunchaku_flux_schnell.py # FLUX.1-schnell deployment (4 steps, fast)
├── modal_nunchaku_qwen.py         # Qwen model deployment
└── README.md                       # This file
```

---

## ⚙️ Model Comparison

| Model | Default Steps | Speed | Quality | Best For |
|-------|---------------|-------|---------|----------|
| **FLUX.1-dev** | 50 | ~60-90s | Excellent | Production, high-quality images |
| **FLUX.1-schnell** | 4 | ~10-15s | Good | Rapid prototyping, previews |

---

## 💡 Tips & Best Practices

1. **First request is slow**: Cold start downloads models (~15GB). Subsequent requests are fast (~10-60s).

2. **Reduce steps for speed**: Use `steps: 20` for faster generation with slightly lower quality.

3. **Use smaller dimensions**: 512×512 generates ~4x faster than 1024×1024.

4. **Set a seed for consistency**: Use the same seed to reproduce exact results.

5. **Prompt tips**:
   - Be specific and descriptive
   - Include style keywords: "photorealistic", "oil painting", "anime style"
   - Mention lighting: "golden hour", "dramatic lighting", "soft focus"

---

## 🔒 Security Notes

- Never commit your `BEARER_TOKEN` or `HF_TOKEN` to version control
- Use Modal secrets for all sensitive credentials
- The bearer token protects your API from unauthorized access


---

## ? Support

If this project helped you, consider buying me a coffee!

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-%E2%98%95-FFDD00?style=flat-square&logo=buymeacoffee&logoColor=black)](https://buymeacoffee.com/mehaksandhudev)

---
---

## 📝 License

This project is provided as-is for educational and personal use. FLUX models are subject to their respective licenses from Black Forest Labs.

---

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request.