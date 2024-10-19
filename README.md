# Cosmos/Geth Snapshot Service

A Go-based service for creating and managing blockchain snapshots from Geth (Ethereum) and Cosmos nodes. The service compresses and stores snapshots periodically and implements automatic retention to delete older snapshots.

## Features

- Automated Geth and Cosmos snapshot creation.
- Configurable retention policy to keep only the latest `n` snapshots.
- Organized storage in subdirectories (`geth/` and `cosmos/`).
- Service management: stop nodes, create snapshots, and restart nodes automatically.
- Simple integration with Nginx for HTTPS serving.

## Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/yourusername/snapshot-service.git
   cd snapshot-service
   ```
2. **Build the project**:
```bash
make snapshot-creator 
make file-server
```
3. **Create `config.yaml` **:
```yaml
#execution config
geth_rpc_url: "http://localhost:8545"
geth_client_name: "geth"
geth_service_name: "geth.service"
geth_data_dir: "/home/ubuntu/.story/geth/iliad/geth/chaindata"
#consensus config
cosmos_rpc_url: "http://localhost:26657/status"
cosmos_client_name: "story"
cosmos_service_name: "story.service"
cosmos_data_dir: "/home/ubuntu/.story/story/data"
#snapshot config
geth_snapshot_type: "pruned"
cosmos_snapshot_type: "pruned"
snapshot_dir: "/home/ubuntu/snapshot-server/public/snapshots"
snapshot_interval_hours: 4
log_file: "/home/ubuntu/snapshot-server/public/snapshot_service.log"
server_port: 8080

```

## Usage 
```bash
#run it in background (locate it using ps aux | grep "snapshot_creator" and kill it)
./bin/snapshot_creator --config ./ &
#run it in background (locate it using ps aux | grep "file_server" and kill it)
./bin/file_server --config ./ & 
```



## Nginx integration 
the server is hosting the files on port 8080, so just create a nginx config with your domain and make it tsl/ssl 
```bash
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;
    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
    }

    location /snapshots/ {
        proxy_pass http://localhost:8080/snapshots/;
    }
}
```



