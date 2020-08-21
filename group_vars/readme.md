#### Deployment variables

| Variable | Type | Definition |
|--|--|--|
| `deploy_lighthouse_beacon` | `boolean` (`true`/`false`) | Deploy a lighthouse beacon |
| `deploy_lighthouse_validator` | `boolean` (`true`/`false`) | Deploy a lighthouse validator |
| `deploy_teku_beacon_w_validator` | `boolean` (`true`/`false`) | Deploy a teku beacon + validator combo |
| `deploy_nimbus_beacon_w_validator` | `boolean` (`true`/`false`) | Deploy a nimbus beacon + validator combo |
| `deploy_prysm_beacon` | `boolean` (`true`/`false`) | Deploy a prysm beacon |
| `deploy_prysm_validator` | `boolean` (`true`/`false`) | Deploy a prysm validator |
| `deploy_netdata_monitoring` | `boolean` (`true`/`false`) | Deploy netdata to monitor node performance |

#### Raspberry Pi Setup

| Variable | Type | Definition |
|--|--|--|
| `ansible_user` | `string` | Default user on Rapsberry Pis (will be `ubuntu` for Ubuntu OS) |
| `copy_local_key` | `string` | Path to local key on host machine (defaults to `$HOME/.ssh/id_rsa.pub`) |
| `sys_packages` | `Array[] string` | List of default packages to install |

#### Kubernetes setup

| Variable | Type | Definition |
|--|--|--|
| `k3s_version` | `string` | k3s version to install |
| `systemd_dir` | `string` | Path to systemd directory |
| `master_ip` | `string` | Valid IP address for master host (Automatically pulled from hosts.ini file) |
| `extra_k3s_server_args` | `string` | Additional arguments for `k3s server` command (Default is `--no-deploy servicelb` since we are using MetalLB as the load balancer) |

#### Replicated Storage

| Variable | Type | Definition |
|--|--|--|
| `storage_solution` | `string` | Storage solution to use (Currently only `glusterfs`) |
| `storage_path` | `string` | Path to disk to use for replicated storage |
| `storage_size` | `int` | Amount in GB to use for each disk |

#### Docker images

| Variable | Type | Definition |
|--|--|--|
| `geth_image_tag` | `string` | Valid container tag/path for geth |
| `lighthouse_image_tag` | `string` | Valid container tag/path for lighthouse |
| `teku_image_tag` | `string` | Valid container tag/path for teku | 
| `nimbus_image_tag` | `string` | Valid container tag/path for nimbus | 
| `prysm_beacon_image_tag` | `string` | Valid container tag/path for prysm beacon | 
| `prysm_validator_image_tag` | `string` | Valid container tag/path for prysm validator |

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
| `lighthouse_keystore_password` | `string` | Password used to unlock your validator |

#### Teku

| Variable | Type | Definition |
|--|--|--|
| `teku_api_port` | `int` | Teku HTTP API port |
| `teku_api_target_port` | `int` | Teku HTTP API port target |
| `teku_port` | `int` | Teku P2P port |
| `teku_target_port` | `int` | Teku P2P port target |
| `teku_keystore_password` | `string` | Password used to unlock your validator |

| Variable | Type | Definition |
|--|--|--|
| `nimbus_api_port` | `int` | nimbus HTTP API port |
| `nimbus_api_target_port` | `int` | nimbus HTTP API port target |
| `nimbus_port` | `int` | nimbus P2P port |
| `nimbus_target_port` | `int` | nimbus P2P port target |
| `nimbus_keystore_password` | `string` | Password used to unlock your validator |

| Variable | Type | Definition |
|--|--|--|
| `prysm_beacon_api_port` | `int` | prysm HTTP API port |
| `prysm_beacon_api_target_port` | `int` | prysm HTTP API port target |
| `prysm_beacon_peer_port` | `int` | prysm P2P port |
| `prysm_beacon_peer_target_port` | `int` | prysm P2P port target |
| `prysm_beacon_peer_port_2` | `int` | prysm P2P port UDP |
| `prysm_beacon_peer_target_port` | `int` | prysm P2P port target |
| `prysm_beacon_keystore_password` | `string` | Password used to unlock your validator |