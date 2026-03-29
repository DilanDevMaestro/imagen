# Nodos Por Red: Comparativa Y Requisitos

Fecha: 2026-03-29

Carpeta de trabajo creada:

- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\bsc`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\node`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\agave`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\solana-node-cli`

Repos descargados:

- `https://github.com/bnb-chain/bsc`
- `https://github.com/base/node`
- `https://github.com/anza-xyz/agave`
- `https://github.com/solana-developers/solana-node-cli`

## Comparacion Rapida

| Red / Repo | Tipo real del repo | Que necesitas si o si | Hardware / almacenamiento documentado | Metodo principal de arranque | Observacion clave |
| --- | --- | --- | --- | --- | --- |
| BNB Smart Chain / `bsc` | Cliente de nodo BSC basado en `geth` | Binario `geth` o build propio, config de red, `genesis/config`, snapshot recomendado, conectividad P2P/RPC | Mainnet: 3 TB libres, SSD/NVMe, 16 vCPU, 64 GB RAM, 5 MB/s. Testnet: 500 GB, 4 vCPU, 16 GB RAM | Binario directo, Docker o Docker image | Sirve para full node, archive, light y bootnode; para validador hay guia aparte |
| Base / `node` | Stack Docker para correr un nodo Base sobre OP Stack | Docker + Docker Compose, endpoint L1 Ethereum RPC, endpoint Beacon, endpoint Beacon Archiver | Minimo: CPU multicore, 32 GB RAM, 64 GB recomendado, NVMe SSD, storage = `2 x chain size + snapshot + 20% buffer` | `docker compose up --build` | No es standalone puro: depende de infraestructura L1 Ethereum |
| Solana / `agave` | Cliente real de validator y RPC node de Solana | Solana CLI/Agave, claves, discos separados para ledger/accounts/snapshots, conectividad publica estable, build o binarios release | Validator: 12c/24t+, 256 GB RAM+, NVMe dedicados. RPC con account indexes: 16c/32t+, 512 GB RAM+ | `agave-validator ...` | Docker no es recomendado para clusters live; para RPC full conviene nodo separado y sin voting |

## Conclusion Ejecutiva

- La comparativa principal ya queda alineada con repos de nodo real para las tres redes: `bsc`, `base/node` y `agave`.
- `solana-node-cli` sigue siendo util, pero como herramienta complementaria de setup y desarrollo, no como cliente validador productivo.
- Para operar Solana en serio, el repo de referencia en esta carpeta pasa a ser `agave`.

## 1. BNB Smart Chain - Repo `bsc`

### Naturaleza del repo

- Cliente oficial de BNB Smart Chain derivado de `go-ethereum`.
- Usa el binario principal `geth`.
- Soporta ejecucion como full node por defecto, archive node, light node y tambien bootnode.
- BSC trabaja con consenso `Proof of Staked Authority (PoSA)` y 21 validators.

### Requisitos de build

- Go `1.24` o superior.
- Compilador C `GCC 5` o superior.
- Comando de build principal:

```bash
make geth
```

- Suite completa:

```bash
make all
```

- Si aparece el error `Caught SIGILL in blst_cgo_init`, el repo indica recompilar con:

```bash
export CGO_CFLAGS="-O -D__BLST_PORTABLE__"
export CGO_CFLAGS_ALLOW="-O -D__BLST_PORTABLE__"
```

### Hardware documentado

#### Mainnet

- VPS o servidor reciente en Mac OS X, Linux o Windows.
- `3 TB` libres de disco a diciembre 2023.
- SSD obligatorio.
- Se recomienda `gp3`, `8k IOPS`, `500 MB/s` de throughput, latencia de lectura menor a `1 ms`.
- Si el nodo arranca con `snap sync`, el repo indica que necesitara `NVMe SSD`.
- `16` cores de CPU.
- `64 GB` de RAM.
- Sugerencias de instancia:
  - AWS `m5zn.6xlarge`
  - AWS `r7iz.4xlarge`
  - Google Cloud `c2-standard-16`
- Conexion de banda ancha con `5 MB/s` de subida y bajada.

#### Testnet

- VPS o servidor reciente en Mac OS X, Linux o Windows.
- `500 GB` de almacenamiento.
- `4` cores de CPU.
- `16 GB` de RAM.

### Requisitos operativos para full node

- Descargar binario precompilado desde releases o compilar desde fuente.
- Descargar archivos de configuracion `mainnet.zip` o `testnet.zip` desde releases.
- Descargar snapshot de chain data desde `bnb-chain/bsc-snapshots`.
- Estructurar los archivos como espera el nodo.

### Comandos documentados de puesta en marcha

#### Binario precompilado

```bash
# Linux
wget $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep geth_linux |cut -d\" -f4)
mv geth_linux geth
chmod -v u+x geth

# MacOS
wget $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep geth_mac |cut -d\" -f4)
mv geth_macos geth
chmod -v u+x geth
```

#### Configs de red

```bash
# mainnet
wget $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep mainnet |cut -d\" -f4)
unzip mainnet.zip

# testnet
wget $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep testnet |cut -d\" -f4)
unzip testnet.zip
```

#### Arranque full node

```bash
./geth --config ./config.toml --datadir ./node --cache 8000 --rpc.allow-unprotected-txs --history.transactions 0 --history.logs 576000
```

#### Variante de alto rendimiento

```bash
./geth --config ./config.toml --datadir ./node --cache 8000 --rpc.allow-unprotected-txs --history.transactions 0 --history.logs 576000 --tries-verify-mode none
```

El repo aclara que `--tries-verify-mode none` mejora performance si importa poco la consistencia de estado.

### Puertos e interfaces documentadas

- HTTP RPC:
  - `--http`
  - `--http.addr`
  - `--http.port` default `8545`
  - `--http.api` default `eth,net,web3`
  - `--http.corsdomain`
- WebSocket RPC:
  - `--ws`
  - `--ws.addr`
  - `--ws.port` default `8546`
  - `--ws.api` default `eth,net,web3`
  - `--ws.origins`
- IPC:
  - `--ipcdisable`
  - `--ipcpath`
- Bootnode example:
  - puerto `30311`

### Seguridad indicada por el repo

- El repo advierte que exponer HTTP/WS sin endurecimiento tiene implicancias de seguridad.
- Advierte explicitamente sobre intentos activos de subvertir nodos con APIs expuestas.
- Tambien advierte que paginas web locales pueden intentar abusar de endpoints expuestos al host.

### Docker segun el repo

- Imagen publicada en `ghcr.io/bnb-chain/bsc`.
- Build docker:

```bash
make docker
```

- En ARM, si quieres imagen `x86_64`, usar:

```bash
docker build --platform linux/amd64 -t bnb-chain/bsc -f Dockerfile .
```

- Antes de levantar el contenedor, el repo exige tener `config.toml` y `genesis.json` de releases.
- Ambos archivos deben montarse en `/bsc/config`.

#### Docker run

```bash
docker run -v $(pwd)/config:/bsc/config --rm --name bsc -it bnb-chain/bsc
```

#### Docker run con override de RPC

```bash
docker run -v $(pwd)/config:/bsc/config --rm --name bsc -it bnb-chain/bsc --http.addr 0.0.0.0 --http.port 8545 --http.vhosts '*' --verbosity 3
```

#### Puertos mostrados en ejemplo Kubernetes

- P2P: `30311`
- RPC: `8545`
- WS: `8546`

### Para armado de nodo BSC

Necesitas contemplar:

1. Binario `geth` compatible con release actual.
2. `config.toml` y `genesis.json` de la red correcta.
3. Snapshot inicial si quieres reducir el tiempo de sync.
4. Disco SSD/NVMe serio, porque el requerimiento de IOPS y throughput es alto.
5. Memoria alta para mainnet.
6. Politica de exposicion RPC/WS endurecida.
7. Diferenciar si vas a operar full node, archive node, bootnode o validador.

## 2. Base - Repo `node`

### Naturaleza del repo

- Repo de Docker builds para correr tu propio nodo en Base.
- Base esta descrita como Ethereum L2 construida sobre OP Stack.
- El repo no es un binario suelto unico: levanta una arquitectura compuesta.
- Clientes soportados:
  - `reth` por defecto
  - `geth`
  - `nethermind`

### Dependencias duras documentadas

- Docker.
- Docker Compose.
- Un Ethereum L1 full node RPC disponible.
- Endpoint L1 Beacon.
- Endpoint L1 Beacon Archiver.

Variables requeridas en `.env.mainnet` o `.env.sepolia`:

- `OP_NODE_L1_ETH_RPC`
- `OP_NODE_L1_BEACON`
- `OP_NODE_L1_BEACON_ARCHIVER`

Tambien aparecen estos parametros operativos:

- `OP_NODE_L1_BEACON_FETCH_ALL_SIDECARS="true"`
- `OP_NODE_L1_RPC_KIND="debug_geth"`
- `OP_NODE_L1_TRUST_RPC="false"`

### Hardware documentado

#### Minimo

- CPU multicore moderna.
- `32 GB` de RAM.
- `64 GB` recomendados.
- Disco `NVMe SSD`.
- Almacenamiento calculado como:

```text
2 * current chain size + snapshot size + 20% buffer
```

#### Especificacion de produccion que usa Base

##### Reth archive node recomendado

- AWS `i7i.12xlarge`
- `RAID 0` con todos los NVMe locales `/dev/nvme*`
- Filesystem `ext4`

##### Geth full node

- AWS `i7i.12xlarge`
- `RAID 0` con todos los NVMe locales `/dev/nvme*`
- Filesystem `ext4`

### Redes soportadas

- Mainnet.
- Testnet.

### Arranque segun el repo

#### Mainnet

```bash
docker compose up --build
```

#### Testnet

```bash
NETWORK_ENV=.env.sepolia docker compose up --build
```

#### Elegir cliente

```bash
CLIENT=reth docker compose up --build
```

#### Testnet con cliente especifico

```bash
NETWORK_ENV=.env.sepolia CLIENT=reth docker compose up --build
```

### Componentes y puertos visibles en `docker-compose.yml`

#### Servicio `execution`

- Build desde `${CLIENT:-geth}/Dockerfile`
- Volumen de datos: `${HOST_DATA_DIR}:/data`
- `HOST_DATA_DIR` default en `.env`: `./${CLIENT}-data`
- Puertos:
  - `8545:8545` RPC
  - `8546:8546` WebSocket
  - `7301:6060` metrics
  - `30303:30303` P2P TCP
  - `30303:30303/udp` P2P UDP

#### Servicio `node`

- Depende de `execution`
- Puertos:
  - `7545:8545` RPC
  - `9222:9222` P2P TCP
  - `9222:9222/udp` P2P UDP
  - `7300:7300` metrics
  - `6060:6060` pprof

### Configuracion relevante de `.env.mainnet`

- Red:
  - `RETH_CHAIN=base`
  - `OP_NODE_NETWORK=base-mainnet`
  - `OP_GETH_OP_NETWORK=base-mainnet`
- Sequencer:
  - `RETH_SEQUENCER_HTTP=https://mainnet-sequencer.base.org`
  - `OP_SEQUENCER_HTTP=https://mainnet-sequencer.base.org`
  - `OP_GETH_SEQUENCER_HTTP=https://mainnet-sequencer.base.org`
  - `OP_RETH_SEQUENCER_HTTP=https://mainnet-sequencer.base.org`
- Sync:
  - `OP_NODE_SYNCMODE=execution-layer`
  - `OP_NODE_VERIFIER_L1_CONFS=4`
  - `OP_NODE_ROLLUP_LOAD_PROTOCOL_VERSIONS=true`
- Engine:
  - `OP_NODE_L2_ENGINE_KIND=reth`
  - `OP_NODE_L2_ENGINE_RPC=ws://execution:8551`
  - `OP_NODE_L2_ENGINE_AUTH=/tmp/engine-auth-jwt`
  - `OP_NODE_L2_ENGINE_AUTH_RAW=...`
- P2P:
  - `OP_NODE_P2P_AGENT=base`
  - `OP_NODE_P2P_LISTEN_IP=0.0.0.0`
  - `OP_NODE_P2P_LISTEN_TCP_PORT=9222`
  - `OP_NODE_P2P_LISTEN_UDP_PORT=9222`
  - `OP_NODE_INTERNAL_IP=true`
  - `OP_NODE_P2P_BOOTNODES=...`
- RPC:
  - `OP_NODE_RPC_ADDR=0.0.0.0`
  - `OP_NODE_RPC_PORT=8545`
- Cache geth:
  - `GETH_CACHE=20480`
  - `GETH_CACHE_DATABASE=20`
  - `GETH_CACHE_GC=12`
  - `GETH_CACHE_SNAPSHOT=24`
  - `GETH_CACHE_TRIE=44`
- Monitoring:
  - `OP_NODE_METRICS_ENABLED=true`
  - `OP_NODE_METRICS_ADDR=0.0.0.0`
  - `OP_NODE_METRICS_PORT=7300`
  - `STATSD_ADDRESS=172.17.0.1`

### Snapshots y opciones extra

- El repo indica que hay snapshots para acelerar sync.
- Remite a `docs.base.org` para links y procedimiento de restore.
- Features opcionales declaradas:
  - EthStats Monitoring
  - Trusted RPC Mode
  - Snap Sync experimental
  - Flashblocks para `reth`
  - Pruning con `RETH_PRUNING_ARGS`

### Reth segun `reth/README.md`

- `reth` puede correr en modo vanilla o Flashblocks.
- Flashblocks depende de `RETH_FB_WEBSOCKET_URL`.
- Si esa variable no existe, corre modo vanilla.
- Si existe, corre con soporte Flashblocks.

### Dependencias de versiones fijadas en el repo

- `base-reth-node`: repo `https://github.com/base/base.git`, tag `v0.6.0`
- `nethermind`: repo `https://github.com/NethermindEth/nethermind.git`, tag `1.36.2`
- `op-geth`: repo `https://github.com/ethereum-optimism/op-geth.git`, tag `v1.101701.0`
- `op-node`: repo `https://github.com/ethereum-optimism/optimism.git`, tag `op-node/v1.16.9`

### Para armado de nodo Base

Necesitas contemplar:

1. Infra Docker funcional.
2. Disco NVMe serio y storage holgado porque depende de chain size y snapshot size.
3. Endpoints confiables de Ethereum L1 RPC, Beacon y Beacon Archiver.
4. Elegir cliente `reth`, `geth` o `nethermind` antes del arranque.
5. Definir si usaras snapshots, trusted RPC, flashblocks o pruning.
6. Abrir puertos de execution y op-node si vas a exponer servicios.
7. Entender que es una arquitectura de L2 con dependencia operacional externa en L1.

## 3. Solana - Repo `agave`

### Naturaleza real del repo

- `agave` es el software real de validator y RPC node para Solana.
- Incluye el binario `agave-validator`.
- La propia documentacion lo posiciona para operar validator o RPC node sobre Devnet, Testnet y Mainnet Beta.
- El README indica claramente que el build debug no es apto para Testnet o Mainnet y que para produccion se debe usar release build o binarios precompilados.

### Instalacion y build

#### Prerrequisitos generales

- Rust, cargo y rustfmt.
- En Linux, dependencias como `libssl-dev`, `pkg-config`, `zlib1g-dev`, `protobuf`, `llvm`, `clang`, `cmake`, `make`, `libprotobuf-dev`, `libclang-dev`, `libudev-dev`.

#### Ubuntu

```bash
sudo apt-get update
sudo apt-get install libssl-dev libudev-dev pkg-config zlib1g-dev llvm clang cmake make libprotobuf-dev protobuf-compiler libclang-dev
```

#### Build local basico

```bash
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup component add rustfmt
git clone https://github.com/anza-xyz/agave.git
cd agave
./cargo build
```

#### Nota critica del repo

- `./cargo build` genera un build debug que no es apto para validator de testnet o mainnet.
- Para uso real, el repo remite a `docs/src/cli/install.md#build-from-source` o a binarios release precompilados.

#### Instalador Agave recomendado

MacOS y Linux:

```bash
sh -c "$(curl -sSfL https://release.anza.xyz/LATEST_AGAVE_RELEASE_VERSION/install)"
```

Windows:

```bash
cmd /c "curl https://release.anza.xyz/LATEST_AGAVE_RELEASE_VERSION/agave-install-init-x86_64-pc-windows-msvc.exe --output C:\agave-install-tmp\agave-install-init.exe --create-dirs"
C:\agave-install-tmp\agave-install-init.exe LATEST_AGAVE_RELEASE_VERSION
```

#### Build from source documentado por la CLI

```bash
./scripts/cargo-install-all.sh .
export PATH=$PWD/bin:$PATH
```

### Requisitos operativos documentados

#### SOL minimo

- No hay minimo estricto de SOL para correr un validator.
- Para participar en consenso necesitas vote account con reserva rent-exempt de `0.02685864 SOL`.
- El voting puede costar hasta `1.1 SOL` por dia.

#### Hardware del validator

- CPU:
  - `2.8 GHz` base clock o superior.
  - Soporte SHA extensions.
  - AMD Gen 3 o superior.
  - Intel Ice Lake o superior.
  - Mejor mayor clock que mas cores.
  - AVX2 para usar binarios oficiales release.
  - AVX512f ayuda.
  - `12 cores / 24 threads` o mas.
- RAM:
  - `256 GB` o mas.
  - ECC sugerida.
  - Motherboard con capacidad de `512 GB` sugerida.
- Discos:
  - NVMe PCIe Gen3 x4 o mejor.
  - Accounts: `1 TB` o mas.
  - Ledger: `1 TB` o mas.
  - Snapshots: `500 GB` o mas.
  - OS opcionalmente en disco separado de `500 GB` o mas.
  - Accounts y ledger pueden compartir disco, pero no se recomienda por IOPS.
  - Para live operation el repo recomienda no juntar accounts y ledger.

#### Hardware adicional para RPC node

- `16 cores / 32 threads` o mas.
- `512 GB` o mas de RAM para usar todos los account indexes.
- Puede requerir ledger mas grande si quieres mas historial transaccional.

### Restricciones operativas fuertes del repo

- Correr un nodo Agave en cloud requiere mucha mas experiencia operacional.
- Docker para clusters live, incluido mainnet-beta, no es recomendado ni generalmente soportado.
- Se recomienda IP publica IPv4 estable.
- Para nodo sin stake: `1 GBit/s` simetrico es suficiente.
- Para nodo con stake: al menos `2 GBit/s` simetrico; `10 GBit/s` recomendado.
- No se recomienda correr detras de NAT.

### Firewall y puertos

- Trafico requerido:
  - TCP y UDP `8000-8030` hacia la IP del validator para P2P.
  - Puede limitarse con `--dynamic-port-range`.
- Opcional RPC:
  - HTTP JSON-RPC `8899`
  - WebSocket JSON-RPC `8900`
- SSH recomendado solo desde IP estatica de administracion.

### Setup de validator documentado

- Instalar localmente Solana CLI y verificar:

```bash
solana --version
agave-validator --version
```

- Configurar cluster, por ejemplo Testnet:

```bash
solana config set --url https://api.testnet.solana.com
solana config get
```

- Generar 3 keypairs:

```bash
solana-keygen new -o validator-keypair.json
solana-keygen new -o vote-account-keypair.json
solana-keygen new -o authorized-withdrawer-keypair.json
```

- Crear vote account desde maquina confiable:

```bash
solana create-vote-account -ut \
    --fee-payer ./validator-keypair.json \
    ./vote-account-keypair.json \
    ./validator-keypair.json \
    ./authorized-withdrawer-keypair.json
```

- El repo insiste en no guardar la clave withdrawer en el servidor remoto del validator.
- El setup recomendado usa un usuario no root, por ejemplo `sol`.
- En la guia de validator se documenta montaje separado para `/mnt/ledger`.

### Setup de RPC node documentado

- Un RPC node usa el mismo proceso base que un validator.
- Primero se monta como validator y luego se agregan flags RPC.
- El repo recomienda que un RPC node tipicamente no vote.
- Tambien advierte que el RPC no debe exponerse directamente a internet sin proteccion.

Flags destacados:

- `--full-rpc-api`
- `--no-voting`
- `--private-rpc`
- `--only-known-rpc`
- `--ledger /mnt/ledger`
- `--accounts /mnt/accounts`
- `--rpc-port 8899`
- `--dynamic-port-range 8000-8020`
- `--limit-ledger-size`

### Clusters y entrypoints documentados

- Devnet:
  - RPC URL `https://api.devnet.solana.com`
  - Gossip `entrypoint.devnet.solana.com:8001`
- Testnet:
  - RPC URL `https://api.testnet.solana.com`
  - Gossip `entrypoint.testnet.solana.com:8001`
- Mainnet Beta:
  - RPC URL `https://api.mainnet-beta.solana.com`
  - Gossip `entrypoint.mainnet-beta.solana.com:8001`

### Ejemplo de comando real de validator

La documentacion de clusters incluye comandos `agave-validator` con estos bloques:

- `--identity`
- multiples `--known-validator`
- `--ledger ledger`
- `--rpc-port 8899`
- multiples `--entrypoint ...:8001`
- `--expected-genesis-hash ...`
- `--wal-recovery-mode skip_any_corrupted_record`
- `--limit-ledger-size`

### Para armado de nodo Solana con `agave`

Necesitas contemplar:

1. Agave release o build release, no debug build.
2. Solana CLI y `agave-validator` instalados y verificados.
3. Claves de identidad, vote account y withdrawer con manejo seguro.
4. Discos NVMe separados para accounts, ledger y snapshots.
5. Mucha RAM si vas a servir RPC pesado o account indexes.
6. Conectividad de muy buen nivel y sin NAT idealmente.
7. Definir si sera validator con voting o RPC node separado con `--no-voting`.
8. Blindar RPC si va a quedar accesible desde terceros.

## 4. Solana Tooling Complementario - Repo `solana-node-cli`

### Rol del repo

- `solana-node-cli` queda como repo complementario para instalar y preparar tooling local.
- No reemplaza a `agave` como software de nodo real.

### Requisito explicito del repo

- `NodeJS >= 22`

El propio codigo fija:

- `MIN_NODE_VERSION = 22.0.0`

### Que instala esta CLI

- Rust y Cargo mediante Rustup.
- Agave CLI tool suite.
- Mucho CLI.
- `solana-verify`.
- Anchor y AVM.
- Yarn como dependencia actual de Anchor.
- Trident Fuzzer.
- Zest.

### Dependencias del sistema detectadas en Linux Debian/Ubuntu

- `build-essential`
- `pkg-config`
- `libudev-dev`
- `llvm`
- `libclang-dev`
- `protobuf-compiler`
- `libssl-dev`

### Uso base

```bash
npx solana --help
```

El propio README indica que no se recomienda instalarlo como paquete npm global.

## Comparativa Operativa Detallada

| Aspecto | BSC `bsc` | Base `node` | Solana `agave` |
| --- | --- | --- | --- |
| Tipo de artefacto | Cliente de blockchain | Stack Docker L2 | Cliente real de validator/RPC |
| Red soportada desde el repo | BNB Smart Chain mainnet/testnet | Base mainnet/testnet | Devnet, Testnet y Mainnet Beta |
| Dependencia externa obligatoria | No explicita una L1 externa | Si, Ethereum L1 RPC + Beacon + Archiver | No una L1 externa, pero si claves, vote account y entrypoints/known validators |
| Hardware explicito en repo | Si, detallado | Si, detallado | Si, detallado |
| Build local | Go 1.24+, GCC 5+ | Docker build por cliente | Rust + toolchain release Agave |
| Snapshot documentado | Si | Si | Si |
| Full node real desde repo | Si | Si | Si |
| Archive / pruning | Si | Si, segun cliente y pruning args | Si, con `--limit-ledger-size` y variantes RPC |
| Bootnode / P2P documentado | Si | Si | Si, con dynamic port range y entrypoints gossip |
| Riesgo de comparacion engañosa | Bajo | Bajo | Bajo |

## Recomendacion Practica

- Si la meta es preparar documentacion de nodos productivos por red, la triada principal de este documento ya queda lista con:
  - BSC: `bsc`
  - Base: `node`
  - Solana: `agave`
- `solana-node-cli` queda como complemento util para entorno local, no como reemplazo del validator.

## Fuentes Revisadas En Los Repos Descargados

- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\bsc\README.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\bsc\docs\docker\README.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\node\README.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\node\reth\README.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\node\.env.mainnet`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\node\.env`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\node\docker-compose.yml`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\node\versions.env`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\agave\README.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\agave\docs\src\cli\install.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\agave\docs\src\operations\requirements.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\agave\docs\src\operations\setup-a-validator.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\agave\docs\src\operations\setup-an-rpc-node.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\agave\docs\src\clusters\available.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\solana-node-cli\README.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\solana-node-cli\package.json`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\solana-node-cli\CHANGELOG.md`
- `C:\Users\maest\Desktop\Genesys-deploys\NODOS\solana-node-cli\src\lib\install.ts`




----------------------------------------------------------------

# Analisis De Esta PC Y Armado Recomendado Para Nodos 2026

Fecha: 2026-03-29

## 1. Especificaciones Detectadas En Esta PC

Analisis realizado con comandos de PowerShell sobre Windows 11.

### Comandos usados

```powershell
$cpu = Get-CimInstance Win32_Processor | Select-Object Name,NumberOfCores,NumberOfLogicalProcessors,MaxClockSpeed,Manufacturer
$ramModules = Get-CimInstance Win32_PhysicalMemory | Select-Object Manufacturer,PartNumber,Capacity,Speed,ConfiguredClockSpeed
$totalRam = [math]::Round(((Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory / 1GB),2)
$board = Get-CimInstance Win32_BaseBoard | Select-Object Manufacturer,Product,SerialNumber
$bios = Get-CimInstance Win32_BIOS | Select-Object SMBIOSBIOSVersion,Manufacturer,ReleaseDate
$os = Get-CimInstance Win32_OperatingSystem | Select-Object Caption,Version,OSArchitecture
$gpu = Get-CimInstance Win32_VideoController | Select-Object Name,AdapterRAM,DriverVersion
$disks = Get-PhysicalDisk | Select-Object FriendlyName,MediaType,BusType,Size,HealthStatus,SpindleSpeed
$volumes = Get-Volume | Where-Object DriveLetter | Select-Object DriveLetter,FileSystemLabel,FileSystem,SizeRemaining,Size
$net = Get-NetAdapter | Where-Object Status -eq 'Up' | Select-Object Name,InterfaceDescription,LinkSpeed,MacAddress
```

### Resultado resumido

- SO: Windows 11 Home 64-bit.
- CPU: AMD Ryzen 7 5700G.
- Cores / threads: 8 / 16.
- Clock maximo reportado: 4.3 GHz.
- RAM total: 31.3 GB.
- RAM instalada: 2 x 16 GB Kingston DDR4-3200.
- Motherboard: ASUS ROG STRIX B550-F GAMING WIFI II.
- GPU: Radeon integrada del 5700G.
- Red fisica: Intel I226-V a 1 Gbps.

### Almacenamiento detectado

- NVMe: `PC SN5000S SDEPNSJ-512G-1006` de 512 GB.
- SATA SSD: `HS-SSD-WAVE(N) 1024G` de 1 TB.

### Espacio visible en volúmenes

- Unidad `T:` de 512 GB con aprox. 486 GB libres.
- Unidad `C:` de 1 TB con aprox. 404 GB libres.
- Espacio libre total visible: aprox. 890 GB.

## 2. Diagnostico Frente A BSC

### Requisito objetivo de BSC mainnet

- 16 cores CPU.
- 64 GB RAM.
- 3 TB libres.
- SSD obligatorio; NVMe preferible y en ciertos escenarios necesario.
- Red de al menos 5 MB/s subida y bajada.

### Estado de esta PC frente a BSC

- CPU: insuficiente.
  - Tienes 8 cores y el repo pide 16 cores para mainnet.
- RAM: insuficiente.
  - Tienes 32 GB y BSC pide 64 GB.
- Disco: muy insuficiente.
  - Tienes 1.5 TB brutos y menos de 1 TB libre visible.
  - BSC pide 3 TB libres solo para mainnet.
- Tipo de disco: insuficiente para una operacion comoda.
  - Solo tienes 512 GB NVMe.
  - El resto es SATA SSD; para BSC serio conviene mucho mas NVMe grande.
- Red: aceptable en enlace local si tu proveedor realmente entrega 1 Gbps estable.
  - No tengo medicion real de internet desde estos comandos, solo el link speed del adaptador.

### Que habria que mejorar para BSC

1. Subir CPU a una plataforma de al menos 16 cores reales.
2. Llevar RAM a 64 GB como piso; 128 GB mejor para margen operativo.
3. Agregar al menos un NVMe de 4 TB dedicado a BSC.
4. Preferir Linux para operacion continua, aunque el repo menciona Windows como posible.
5. Mantener la red cableada y estable.

## 3. Diagnostico Frente A Base

### Requisito objetivo de Base

- CPU multicore moderna.
- 32 GB RAM minimo.
- 64 GB recomendado.
- NVMe SSD.
- Storage calculado como `2 x chain size + snapshot + 20% buffer`.
- Docker + Docker Compose.
- Ademas necesitas RPC L1 Ethereum, Beacon y Beacon Archiver.

### Estado de esta PC frente a Base

- CPU: usable pero justa.
  - El 5700G podria levantar algo de laboratorio, pero no es una plataforma ideal para operar Base con holgura.
- RAM: en el minimo, no en el recomendado.
  - Tienes 32 GB, que coincide con el minimo.
  - Para produccion o coexistencia con otros servicios, yo no lo consideraria suficiente.
- Disco: insuficiente.
  - El cuello de botella mas claro es storage NVMe.
  - Tienes 512 GB NVMe y 1 TB SATA; Base prefiere NVMe y bastante margen de capacidad.
- Dependencia externa: faltante operacional.
  - Necesitas endpoints L1 Ethereum adicionales; no basta con la PC sola.

### Que habria que mejorar para Base

1. Llevar RAM a 64 GB minimo recomendable.
2. Agregar uno o dos NVMe grandes, minimo 2 TB y preferible 4 TB segun el crecimiento.
3. Mantener Base separado de otras cargas intensivas.
4. Contar con servicios L1 Ethereum confiables.

## 4. Puede Esta PC Correr BSC Y Base A La Vez

Respuesta corta: no, no de forma seria.

### Motivos

- CPU insuficiente para correr BSC mainnet con margen y ademas Base.
- 32 GB RAM no alcanzan para ambas cargas con estabilidad.
- El almacenamiento es el bloqueo principal:
  - BSC solo ya pide 3 TB libres.
  - Base ademas necesita NVMe y bastante crecimiento.
- Solo hay 1 NVMe pequeño de 512 GB.
- El segundo SSD es SATA, que no es lo ideal para esta combinacion.

## 5. Upgrade Minimo Sobre Esta PC

Si quisieras exprimir esta misma plataforma B550 sin cambiar motherboard, el maximo upgrade razonable seria:

- CPU: Ryzen 9 5950X.
- RAM: 128 GB DDR4 si la placa y los modulos elegidos lo soportan estable.
- Discos:
  - NVMe 4 TB dedicado a Base o BSC.
  - SSD adicional 4 TB, idealmente tambien NVMe si usas expansion PCIe.

### Limite de este enfoque

- Aun con 5950X + 128 GB + discos nuevos, seguiria siendo una plataforma apretada para BSC + Base juntos en serio.
- Para Solana full node directamente no alcanza.
- Para BSC solo podria quedar bastante mejor.
- Para Base solo podria quedar razonable.
- Para ambos a la vez, no es la opcion que te recomendaria.

## 6. PC Recomendada 2026 Para Correr BSC Y BASE En Una Sola PC

Objetivo de esta build:

- correr BSC mainnet y Base en la misma workstation
- separar discos por carga
- dejar margen de crecimiento 2026+
- operar estable 24/7

### Build recomendada

| Componente | Recomendacion | Motivo |
| --- | --- | --- |
| CPU | AMD Threadripper Pro 7975WX o 7965WX | Muchos cores, alto rendimiento sostenido, mucha expansion PCIe y plataforma de workstation real |
| Motherboard | WRX90 de gama workstation | Necesitas muchas lineas PCIe, varios NVMe y gran capacidad de RAM ECC |
| RAM | 256 GB DDR5 ECC RDIMM | BSC + Base juntos merecen mucho margen; ECC ayuda en cargas 24/7 |
| Disco SO | NVMe 2 TB Gen4 o Gen5 | Sistema, Docker, herramientas, logs y utilidades |
| Disco BSC | NVMe enterprise 8 TB | BSC ya pide 3 TB libres y va a seguir creciendo |
| Disco Base | NVMe 4 TB | Espacio razonable para Base con snapshots y crecimiento |
| Disco snapshots / scratch | NVMe 4 TB | Acelera restores, staging y mantenimiento |
| Red | NIC 10GbE Intel o Mellanox | Aunque hoy tu internet no sea 10G, te da piso serio para I/O y futuro |
| GPU | Basica o integrada, sin foco gaming | Para nodos no es prioridad |
| PSU | 1200W 80+ Platinum | Plataforma workstation + varios NVMe + estabilidad 24/7 |
| Refrigeracion | AIO 360 mm o aire premium workstation | Carga sostenida, ruido controlado y estabilidad |
| Gabinete | Full tower con alto flujo de aire | Muchos discos, buena ventilacion, mantenimiento mas limpio |
| UPS | 1500VA o superior senoidal | Fundamental para cortes y evitar corrupcion de datos |

### Distribucion recomendada de cargas

- BSC en su propio NVMe de 8 TB.
- Base `execution` y `op-node` en NVMe de 4 TB dedicado.
- SO y herramientas separados.
- Snapshots y restauraciones en otro NVMe distinto.

### Sistema operativo recomendado

- Ubuntu LTS.

### Comentario tecnico

- Si quieres una sola PC para ambas redes, el verdadero cuello de botella es storage y no solo CPU.
- Por eso en esta build sobredimensiono NVMe y RAM.
- Esta build es de workstation seria, no de PC gamer adaptada.

## 7. PC Recomendada 2026 Para Full Nodo Solana

Objetivo de esta build:

- full node Solana serio
- margen para validator o RPC node fuerte
- storage separado segun las guias de Agave
- conectividad y memoria coherentes con el perfil de Solana

### Build recomendada

| Componente | Recomendacion | Motivo |
| --- | --- | --- |
| CPU | AMD Threadripper Pro 7965WX o 7975WX | Solana necesita clock alto, muchos hilos, gran I/O y mucha RAM direccionable |
| Motherboard | WRX90 workstation | Soporte de ECC, varios NVMe y plataforma mas apropiada que escritorio comun |
| RAM | 512 GB DDR5 ECC RDIMM | Agave indica 256 GB para validator y 512 GB para RPC con account indexes |
| Disco SO | NVMe 2 TB | SO, herramientas y monitoreo |
| Disco accounts | NVMe enterprise 2 TB minimo, mejor 4 TB | Accounts merece disco dedicado y rapido |
| Disco ledger | NVMe enterprise 2 TB minimo, mejor 4 TB | Ledger separado reduce cuellos de IOPS |
| Disco snapshots | NVMe 1 TB minimo, mejor 2 TB | Restore y rotacion de snapshots |
| Red | 10GbE real como piso de diseño | El repo sugiere 2 Gbps para stake y 10 Gbps recomendado |
| PSU | 1200W 80+ Platinum | Estabilidad 24/7 |
| Refrigeracion | Solucion premium de workstation | Carga sostenida continua |
| Gabinete | Full tower de alto flujo | Temperaturas y espacio |
| UPS | 1500VA o superior senoidal | Muy recomendable para no corromper ledger/accounts |

### Si el objetivo es solo validator sin RPC pesado

- Podrias bajar a 256 GB ECC RAM.
- Podrias bajar CPU a una opcion algo menor, pero no recomiendo apretar demasiado si quieres vida util 2026+.

### Si el objetivo es RPC node pesado

- Mantendria 512 GB ECC RAM.
- Mantendria discos separados y mas grandes.
- Mantendria 10GbE y plataforma workstation.

## 8. Comparativa De Armado 2026

| Perfil | CPU | RAM | Storage | Red | Comentario |
| --- | --- | --- | --- | --- | --- |
| Tu PC actual | Ryzen 7 5700G | 32 GB DDR4 | 512 GB NVMe + 1 TB SATA | 1GbE | No apta para BSC mainnet ni para BSC+Base juntos; muy lejos de Solana full node |
| Build para BSC + Base juntos | Threadripper Pro 7965WX/7975WX | 256 GB ECC | 2 TB SO + 8 TB BSC + 4 TB Base + 4 TB scratch | 10GbE | Build workstation seria y equilibrada para ambas redes en una sola maquina |
| Build para Solana full node | Threadripper Pro 7965WX/7975WX | 512 GB ECC | 2 TB SO + 2-4 TB accounts + 2-4 TB ledger + 1-2 TB snapshots | 10GbE | Build enfocada a validator/RPC serio con Agave |

## 9. Recomendacion Final Para Tu Caso

### Si quieres gastar lo minimo

- Usa esta PC solo para laboratorio, pruebas o Base muy contenido.
- No la usaria para BSC mainnet serio.
- No la usaria para Solana full node.

### Si quieres una sola maquina para BSC + Base

- Salta directo a plataforma workstation con Threadripper Pro, 256 GB ECC y varios NVMe dedicados.

### Si quieres Solana serio

- Haz otra maquina separada.
- Solana compite muy fuerte por RAM, IOPS y red.
- Mezclar Solana con BSC o Base en la misma PC no es buena idea si buscas estabilidad real.

## 10. Veredicto Sobre Esta PC

### Sirve para:

- desarrollo
- testing
- snapshots parciales
- tooling
- laboratorio Docker
- Base muy contenido y no ideal

### No sirve bien para:

- BSC mainnet serio
- BSC + Base juntos
- Solana full node real

### Cuellos de botella principales

1. RAM.
2. Capacidad de NVMe.
3. Cantidad total de storage.
4. Plataforma no workstation para cargas multidisco / mucha RAM / ECC.




-------------------------------------------------

# Lista De Compra Exacta Para Nodos 2026

Fecha: 2026-03-29

Este documento complementa [NODOS/ANALISIS_PC_PARA_NODOS_2026.md](NODOS/ANALISIS_PC_PARA_NODOS_2026.md) y te deja referencias de compra concretas.

## Criterio De Compatibilidad Que Si Verifique

- Tu placa actual es `ASUS ROG STRIX B550-F GAMING WIFI II`, plataforma AM4.
- `ASUS ProArt X870E-CREATOR WIFI` usa socket `AM5`, DDR5 UDIMM, hasta `256 GB`, 4 x M.2 y 10GbE integrado.
- `ASUS Pro WS WRX90E-SAGE SE` usa socket `sTR5`, soporta `Ryzen Threadripper PRO 7000/9000 WX`, `8 x DIMM DDR5 ECC Registered`, `4 x M.2`, `2 x 10GbE` integrados y formato `EEB`.
- `Fractal Meshify 2 XL` soporta `SSI-EEB`, `E-ATX` y fuente `ATX`, por lo que sirve para la WRX90.
- `Micron MTC40F2046S1RC64BR` aparece como memoria compatible para `ASUS Pro WS WRX90E-SAGE SE` en el configurador de Crucial.
- `Micron MTC40F2047S1RC56BR` aparece como memoria compatible de `128 GB DDR5-5600 RDIMM` para `ASUS Pro WS WRX90E-SAGE SE`.
- `Samsung 9100 PRO` aparece como SSD M.2 NVMe Gen5 vigente en 2026.
- `Samsung 990 PRO` sigue siendo referencia M.2 NVMe Gen4 vigente.

## 1. Referencia Exacta Para Aprovechar Tu PC Actual

Objetivo:

- dejar tu AM4 al maximo razonable
- laboratorio fuerte
- Base contenido
- BSC no productivo serio, pero bastante mejor que hoy

### BOM exacta sobre tu placa actual

| Componente | Modelo exacto | Compatibilidad |
| --- | --- | --- |
| CPU | AMD Ryzen 9 5950X | Compatible con plataforma AM4/B550 |
| Cooler | Noctua NH-D15 chromax.black | Compatible con AM4 |
| RAM | Corsair Vengeance LPX 128GB (4x32GB) DDR4-3200 CL16, modelo `CMK128GX4M4E3200C16` | DDR4 UDIMM estándar para AM4; apunta al máximo práctico de 128 GB |
| SSD NVMe principal | Samsung 990 PRO 4TB M.2 NVMe 2280 | Formato M.2 2280 estándar |
| SSD SATA secundario | Samsung 870 EVO 4TB 2.5" SATA | Compatible con puertos SATA del B550 |
| PSU | Seasonic Focus GX-1000 ATX 3.0 | Compatible con ATX estándar |

### Qué esperar de esta ruta

- Mucho mejor que tu PC actual.
- Aún no la consideraría una máquina seria para `BSC mainnet`.
- Puede servir para `Base` de laboratorio o para pruebas prolongadas.
- No la usaría para `Solana full node`.

## 2. PC Exacta 2026 Para Correr BSC Y BASE En Una Sola Maquina

Objetivo:

- una sola workstation
- BSC mainnet + Base al mismo tiempo
- storage separado por carga
- red integrada de alto nivel
- margen real 2026+

## Opcion recomendada principal

### BOM exacta

| Componente | Modelo exacto | Por qué |
| --- | --- | --- |
| CPU | AMD Ryzen Threadripper PRO 7965WX | 24 cores, plataforma workstation real, margen amplio para dos nodos pesados |
| Motherboard | ASUS Pro WS WRX90E-SAGE SE | sTR5, 8 DIMM DDR5 ECC RDIMM, 4 x M.2, dual 10GbE, formato workstation |
| Cooler CPU | Noctua NH-U14S TR5-SP6 | Diseñado específicamente para TR5/SP6 |
| RAM | 4 x Micron 64GB DDR5-6400 RDIMM `MTC40F2046S1RC64BR` | 256 GB totales; módulo compatible detectado para esta placa |
| SSD SO / herramientas | Samsung 990 PRO 4TB M.2 NVMe 2280 | SO, Docker, logs, utilidades |
| SSD BSC | Samsung 9100 PRO 8TB M.2 NVMe 2280 | Disco dedicado para BSC y su crecimiento |
| SSD Base | Samsung 990 PRO 4TB M.2 NVMe 2280 | Disco dedicado para Base |
| SSD snapshots / scratch | Samsung 990 PRO 4TB M.2 NVMe 2280 | Restore, staging, snapshots y operaciones |
| Gabinete | Fractal Design Meshify 2 XL `FD-C-MES2X-01` | Soporta SSI-EEB y ATX; encaja la WRX90 |
| PSU | Seasonic PRIME TX-1600 ATX 3 | Margen sobrado para workstation 24/7 y muchos discos |
| UPS | APC Smart-UPS SMT1500IC | Protección de energía seria para storage intensivo |
| SO recomendado | Ubuntu 24.04 LTS | Más razonable para operar nodos 24/7 |

### Compatibilidad de esta build

- CPU y motherboard:
  - `7965WX` figura en soporte CPU de la `Pro WS WRX90E-SAGE SE`.
- RAM y motherboard:
  - la WRX90 exige `DDR5 ECC Registered Memory`.
  - `Micron MTC40F2046S1RC64BR` figura como memoria compatible para esa placa.
- Case y motherboard:
  - `Meshify 2 XL` soporta `SSI-EEB`, y la WRX90 es `EEB 12 x 13`.
- PSU y case:
  - el case admite PSU ATX.
- SSDs y motherboard:
  - la WRX90 soporta 4 slots `M.2 Key M` tipo `2242/2260/2280/22110`.
  - `Samsung 9100 PRO` y `990 PRO` son M.2 2280 NVMe.

### Distribucion recomendada

- `M.2_1`: SO + herramientas = `Samsung 990 PRO 4TB`
- `M.2_2`: BSC = `Samsung 9100 PRO 8TB`
- `M.2_3`: Base = `Samsung 990 PRO 4TB`
- `M.2_4`: snapshots / scratch = `Samsung 990 PRO 4TB`

### Comentario tecnico

- Esta es la build que yo compraría si de verdad quieres `BSC + Base` en una sola caja.
- Usa plataforma workstation para evitar quedarte corto con RAM, PCIe y storage.

## Opcion mas barata, pero menos holgada

### BOM exacta AM5

| Componente | Modelo exacto | Por qué |
| --- | --- | --- |
| CPU | AMD Ryzen 9 9950X | 16C/32T, muy buen clock, plataforma AM5 actual |
| Motherboard | ASUS ProArt X870E-CREATOR WIFI | AM5, hasta 256 GB DDR5, 4 x M.2, 10GbE integrado |
| Cooler CPU | Noctua NH-D15 G2 | Compatibilidad AM5 y disipación alta |
| RAM | 4 x Crucial 32GB DDR5-5600 UDIMM `CT32G56C46U5` | 128 GB totales, módulo compatible detectado para esta placa |
| SSD SO / herramientas | Samsung 990 PRO 4TB M.2 NVMe 2280 | Sistema y contenedores |
| SSD BSC | Samsung 9100 PRO 8TB M.2 NVMe 2280 | BSC dedicado |
| SSD Base | Samsung 990 PRO 4TB M.2 NVMe 2280 | Base dedicado |
| SSD snapshots / scratch | Samsung 990 PRO 4TB M.2 NVMe 2280 | Operación y staging |
| Gabinete | Fractal Design Meshify 2 XL `FD-C-MES2X-01` | Mucho espacio y flujo de aire |
| PSU | Seasonic PRIME TX-1300 ATX 3 | De sobra para esta build |

### Advertencia de esta opcion

- Es más barata y fácil de armar.
- Pero para `BSC + Base` simultáneos sigue siendo más justa que la WRX90.
- Los `128 GB` son utilizables, pero ya no me parece una build tan cómoda para crecer.

## 3. PC Exacta 2026 Para Full Nodo Solana

Objetivo:

- full node serio con Agave
- base fuerte para validator y RPC node
- 512 GB ECC
- discos separados como recomienda la documentación

### BOM exacta

| Componente | Modelo exacto | Por qué |
| --- | --- | --- |
| CPU | AMD Ryzen Threadripper PRO 9975WX | CPU 2026 más actual en esa plataforma, 32 cores, soporte oficial WRX90 |
| Motherboard | ASUS Pro WS WRX90E-SAGE SE | Plataforma workstation correcta para 512 GB ECC y varios NVMe |
| Cooler CPU | Noctua NH-U14S TR5-SP6 | Referencia específica para socket TR5/SP6 |
| RAM | 8 x Micron 64GB DDR5-6400 RDIMM `MTC40F2046S1RC64BR` | 512 GB totales, tipo exacto que la placa soporta y Crucial lista como compatible |
| SSD SO | Samsung 990 PRO 4TB M.2 NVMe 2280 | Sistema, herramientas y monitoreo |
| SSD accounts | Samsung 9100 PRO 4TB M.2 NVMe 2280 | Disco dedicado a accounts |
| SSD ledger | Samsung 9100 PRO 4TB M.2 NVMe 2280 | Disco dedicado a ledger |
| SSD snapshots | Samsung 990 PRO 4TB M.2 NVMe 2280 | Snapshots y recuperación |
| Gabinete | Fractal Design Meshify 2 XL `FD-C-MES2X-01` | Acepta SSI-EEB y hardware grande |
| PSU | Seasonic PRIME TX-1600 ATX 3 | Mucho margen para operación continua |
| UPS | APC Smart-UPS SMT1500IC | Muy recomendable para evitar corrupción por cortes |
| SO recomendado | Ubuntu 24.04 LTS | Lo más sensato para operar Agave 24/7 |

### Compatibilidad de esta build

- CPU y motherboard:
  - `9975WX` figura en la lista de CPUs soportados para la WRX90.
- RAM y motherboard:
  - la placa exige `DDR5 ECC Registered Memory`.
  - `MTC40F2046S1RC64BR` aparece como compatible en el configurador para esa placa.
- Storage:
  - todos los SSD elegidos son `M.2 2280 NVMe`, formato soportado por la WRX90.
- Case:
  - `Meshify 2 XL` soporta `SSI-EEB`, así que el board entra.

### Distribucion recomendada para Solana

- `M.2_1`: SO = `Samsung 990 PRO 4TB`
- `M.2_2`: accounts = `Samsung 9100 PRO 4TB`
- `M.2_3`: ledger = `Samsung 9100 PRO 4TB`
- `M.2_4`: snapshots = `Samsung 990 PRO 4TB`

### Si quieres bajar costo sin romper la idea

- CPU alternativa: `Threadripper PRO 7965WX`
- RAM alternativa validator-only: `4 x 64GB = 256 GB`

Pero si tu meta es `full node Solana serio`, dejaría la build en `512 GB`.

## 4. Resumen Rapido De Compra

| Escenario | Compra exacta recomendada |
| --- | --- |
| Exprimir tu PC actual | `5950X + NH-D15 + 128GB DDR4 + 990 PRO 4TB + 870 EVO 4TB` |
| BSC + Base en una sola máquina | `7965WX + WRX90E-SAGE SE + 256GB ECC RDIMM + 9100 PRO 8TB + 3x 990 PRO 4TB + Meshify 2 XL + TX-1600` |
| Solana full node | `9975WX + WRX90E-SAGE SE + 512GB ECC RDIMM + 2x 9100 PRO 4TB + 2x 990 PRO 4TB + Meshify 2 XL + TX-1600` |

## 5. Mi Recomendacion Final

- Si quieres comprar una sola maquina seria: ve por la build `WRX90 + 7965WX + 256GB ECC` para `BSC + Base`.
- Si además vas a meterte con `Solana` en serio: haz otra máquina separada con `WRX90 + 9975WX + 512GB ECC`.
- Si solo quieres laboratorio con bajo gasto: actualiza tu AM4 con `5950X`, pero no la confundas con una estación real de nodos productivos.

## 6. Fuentes De Compatibilidad Usadas

- `ASUS Pro WS WRX90E-SAGE SE` CPU support y tech specs.
- `ASUS ProArt X870E-CREATOR WIFI` tech specs.
- `Crucial compatible upgrade for ASUS Pro WS WRX90E-SAGE SE`.
- `Crucial compatible upgrade for ASUS ProArt X870E-CREATOR WIFI`.
- `Fractal Meshify 2 XL` compatibility page.
- `Samsung Memory Storage` family page mostrando `9100 PRO` como SSD actual.



