# Layerzero OFT from Solana to EVM Step-by-Step Guide
## Requirements

- Ubuntu 24.04.1 LTS (WSL)
- Docker (Latest)
- Rust `v1.75.0`
- Anchor `v0.29`
- Solana CLI `v1.17.31`
- Docker
- Node.js

## Setup
Ubuntu 24.04.1 LTS: https://apps.microsoft.com/detail/9NZ3KLHXDJP5?hl=neutral&gl=KR&ocid=pdpshare

Windows용 Docker 최신 버전: https://docs.docker.com/get-started/get-docker/

### 필수 환경 설정

```bash
sudo apt update
sudo apt upgrade
sudo apt install git vim build-essential
```

### Docker 작동 확인

```bash
docker --version
```

### Rust 설치

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
. "$HOME/.cargo/env"
```

### Solana SDK 설치

```bash
sh -c "$(curl -sSfL https://release.anza.xyz/v1.17.31/install)"
```

### Anchor 설치

0.29.0 버전을 바로 설치하려고 하면 종속성 문제로 오류가 발생하므로 최신 버전을 먼저 설치 후 0.29.0 버전을 이어서 설치해야 함

먼저 0.31.1 버전 설치 후,
```bash
cargo install --git https://github.com/coral-xyz/anchor --tag v0.31.1 anchor-cli --locked
```

0.29.0 버전을 이어서 설치하기
```bash
cargo install --git https://github.com/coral-xyz/anchor --tag v0.29.0 anchor-cli --locked
```

### pnpm 설치

```bash
curl -fsSL https://get.pnpm.io/install.sh | sh -
source ~/.bashrc
```

### 필수 요소 설치

```bash
git clone https://github.com/LayerZero-Labs/devtools.git
cd ~/devtools
```

(05.27) 최근 커밋으로 인해 오류가 발생하므로 과거 브랜치 사용 (필요할 경우에만)  
```bash
git checkout eb6d8df
```

이어서 pnpm을 사용해 구성요소 설치 & env 파일 복사   
```bash
cd examples/oft-solana
pnpm install
cp .env.example .env
```

### .env 파일 작성 

```bash
MNEMONIC=
PRIVATE_KEY= # Private key for EVM contract owner/delegate
SOLANA_PRIVATE_KEY= # 솔라나 버너 지갑 프라이빗 키 입력
SOLANA_KEYPAIR_PATH=
RPC_URL_SOLANA=https://rpc.ankr.com/solana/*** # RPC 서버 주소 입력
RPC_URL_SOLANA_TESTNET=
```

### OFT.json 파일 작성

```bash
mkdir deployments/solana-mainnet
vim deployments/solana-mainnet/OFT.json
```

아래는 Huma Finance (HUMA1821qVDKta3u2ovmfDQeW2fSQouSKE8fkF44wvGw) 기준으로 작성된 샘플 코드입니다.  
[`registerOApp` 트랜잭션](https://solscan.io/tx/2N5XdL59wRMaowNsMjiH3E9bTwuDxfVk53t4GkLznjctswRpqHGgGai5eVV4cZNKHjhhygEJT8S1GACt1R4QVrLg)을 참고하여 작성합니다.  

![image](https://github.com/user-attachments/assets/f480528a-934a-499a-9867-88254e493394)

```bash
{
    "programId": "BRGv9mxfDeZWxJWPUfX687r9f4Kmg1Qhhq5C9cBSKaM",
    "mint": "HUMA1821qVDKta3u2ovmfDQeW2fSQouSKE8fkF44wvGw",
    "mintAuthority": "dSeocUVUmgsnQW2mgK6ywsvR9oDF3TANdohBsrQqGkd",
    "escrow": "Eae9HsnubBNCJiTyMBVhoiynbRBqpD4GbFcPWwscNwRQ",
    "oftStore": "dSeocUVUmgsnQW2mgK6ywsvR9oDF3TANdohBsrQqGkd"
}
```

### 트랜잭션 실행
아래는 Huma Finance (HUMA1821qVDKta3u2ovmfDQeW2fSQouSKE8fkF44wvGw) 기준으로 작성된 샘플 코드입니다.  
```bash
npx hardhat lz:oft:send --amount 1 --src-eid 30168 --dst-eid 30102 \
--oft-address dSeocUVUmgsnQW2mgK6ywsvR9oDF3TANdohBsrQqGkd \
--oft-program-id BRGv9mxfDeZWxJWPUfX687r9f4Kmg1Qhhq5C9cBSKaM \
--to 0x0000000000000000000000008520feed7cdc4905b7b5c3986b19642a456ce983 --compute-unit-price-scale-factor 10
```

- to: 수취인 주소 (EVM: bytes20, Solana: base58)  
- amount: 보낼 토큰 갯수 (1개 = 1)  
- src-eid: 보낼 체인 ID (30168 : Solana Mainnet)  
- dst-eid: 받는 체인 ID (30102 : BSC)  
- oft-address: OFT 주소 (=oftStore / EVM: bytes20, Solana: base58)  
- oft-program-id: OFT 프로그램 ID (=programId, Solana only)

## Docs
Reference: https://github.com/LayerZero-Labs/devtools/blob/main/examples/oft-solana
Chain ID List: https://docs.layerzero.network/v2/deployments/deployed-contracts

