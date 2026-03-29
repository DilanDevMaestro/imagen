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
