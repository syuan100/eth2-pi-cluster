#### Deployment variables

| Variable | Type | Definition |
|--|--|--|
| `deploy_lighthouse_beacon` | `boolean` (`true`/`false`) | Deploy a lighthouse beacon |
| `deploy_lighthouse_validator` | `boolean` (`true`/`false`) | Deploy a lighthouse validator |

#### Raspberry Pi Setup

| Variable | Type | Definition |
|--|--|--|
| `target_user` | `string` | Default user on Rapsberry Pis (will be `ubuntu` for Ubuntu OS) |
| `copy_local_key` | `string` | Path to local key on host machine (defaults to `$HOME/.ssh/id_rsa.pub`) |
| `sys_packages` | `Array[] string` | List of default packages to install |

#### Kubernetes setup

| Variable | Type | Definition |
|--|--|--|
| `k3s_version` | `string` | k3s version to install |
| `systemd_dir` | `string` | Path to systemd directory |
| `master_ip` | `string` | Valid IP address for master host (Automatically pulled from hosts.ini file) |
| `extra_k3s_server_args` | `string` | Additional arguments for `k3s server` command |

#### Replicated Storage

| Variable | Type | Definition |
|--|--|--|
| `storage_solution` | `string` | Storage solution to use (Currently only `glusterfs`) |
| `storage_path` | `string` | Path to disk to use for replicated storage |
|  `storage_size` | `int` | Amount in GB to use for each disk |

#### Docker images

| Variable | Type | Definition |
|--|--|--|
| `geth_image_tag` | `string` | Valid container tag/path for geth |
| `lighthouse_image_tag` | `string` | Valid container tag/path for lighthouse |

*NOTE: Images must be built for arm64 architectures in order to deploy correctly*

#### Geth 

| Variable | Type | Definition |
|--|--|--|
| `geth_json_rpc_port` | `int` | geth container JSON RPC port (defaults to 8545) |
| `geth_json_rpc_port_target` | `int` | geth service JSON RPC port (defaults to 8545) |
| `geth_port` | `int` | Main geth container port (defaults to 30303) |
| `geth_port_target` | `int` | Main geth service port (defaults to 30303) |
| `external_eth1` | `boolean` (`true`/`false`) | Use internal kubernetes Geth instance |
| `eth1_endpoint` | `string` | Eth 1.0 JSON RPC endpoint (defaults to internal kubernetes hostname) |

#### Lighthouse

| Variable | Type | Definition |
|--|--|--|
| `lighthouse_beacon_api_port` | `int` | Lighthouse beacon container HTTP api port (Defaults to 5052) |
| `lighthouse_beacon_api_target_port` | `int` | Lighthouse beacon service HTTP api port (Defaults to 5052) |
| `lighthouse_beacon_wss_port` | `int` | Lighthouse beacon container wss port (Defaults to 5053) |
| `lighthouse_beacon_wss_target_port` | `int` | Lighthouse beacon service wss port (Defaults to 5053) |
| `lighthouse_beacon_port` | `int` | General lighthouse beacon container port (Defaults to 9000) |
| `lighthouse_beacon_target_port` | `int` | General lighthouse beacon service port (Defaults to 9000) |
| `lighthouse_beacon_endpoint` | `string` | Eth 2.0 Beacon HTTP API endpoint (defaults to internal kubernetes hostname) |