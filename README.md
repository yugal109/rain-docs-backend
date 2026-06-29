# MNIST Predict API

Rain HTTP server for the docs playground. Loads trained weights once and serves `POST /api/predict`.

## Layout

```
backend/
  server.rn              # entry — CORS, middleware, listen
  lib/
    config.rn            # PORT
    model.rn             # weights + predict(pixels)
  middleware/
    logger.rn            # request logging
  routes/
    home.rn              # GET /
    health.rn            # GET /api/health
    predict.rn           # POST /api/predict
    register.rn          # wire routes to handlers
  weights/               # w1.bin, b1.bin, w2.bin, b2.bin
```

Modules load with `benutzen "path/to/file.rn"`; the global name is the file stem (e.g. `lib/model.rn` → `model::predict()`).

## Setup

1. Build the Rain VM (sibling `rainc/`):

```bash
cd ../rainc && make
```

2. Copy weight files into `weights/`:

```
weights/w1.bin
weights/b1.bin
weights/w2.bin
weights/b2.bin
```

3. Start the server from this directory:

```bash
make run
# or: ../rainc/rain server.rn
```

Listens on `http://localhost:8080`.

## Deploy on AWS (nginx → Rain)

Rain binds to `8080` on localhost. Put nginx in front on port `80` (or `443` with TLS).

1. Start the Rain server (from this directory):

```bash
make run
# or: rain server.rn
```

Keep it running with systemd, tmux, or screen in production.

2. Install nginx and copy the site config:

```bash
# Amazon Linux
sudo dnf install -y nginx
sudo cp nginx/rain-api.conf /etc/nginx/conf.d/rain-api.conf

# Ubuntu
sudo apt install -y nginx
sudo cp nginx/rain-api.conf /etc/nginx/sites-available/rain-api.conf
sudo ln -sf /etc/nginx/sites-available/rain-api.conf /etc/nginx/sites-enabled/rain-api.conf
```

3. Edit `server_name` in the config (your domain or EC2 public IP), then:

```bash
sudo nginx -t
sudo systemctl enable nginx
sudo systemctl reload nginx
```

4. EC2 security group: allow inbound **80** (and **443** if you add TLS).

5. Point the docs playground at the public URL:

```
NEXT_PUBLIC_API_URL=http://<ec2-ip-or-domain>
```

Routes proxied to Rain:

| Path | Handler |
|------|---------|
| `GET /` | home |
| `GET /api/health` | health check |
| `POST /api/predict` | MNIST predict |

## API

### `POST /api/predict`

Request body:

```json
{
  "pixels": [0, 0, 255, ...]
}
```

- `pixels`: array of **784** ints, row-major 28×28, values **0–255**

Response:

```json
{ "digit": 7 }
```

### `GET /api/health`

```json
{ "status": "ok" }
```
