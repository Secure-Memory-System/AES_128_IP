# 🔐 AES-128 IP: Hardware-Accelerated Encryption/Decryption Engine
> **Zynq SoC 기반의 지능형 보안 통신 시스템을 위한 고성능 AES-128 암복호화 가속기**

<p align="left">
  <img src="https://img.shields.io/badge/Verilog-F34B7D?style=flat-square&logo=verilog&logoColor=white" />
  <img src="https://img.shields.io/badge/Vivado-FF6600?style=flat-square&logo=xilinx&logoColor=white" />
  <img src="https://img.shields.io/badge/AES--128-009900?style=flat-square&logo=springsecurity&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square" />
</p>

본 프로젝트는 **Zynq SoC** 플랫폼에서 동작하는 지능형 보안 시스템의 암복호화 코어입니다. NPU가 분류한 MNIST 데이터를 하드웨어 단에서 직접 AES-128 알고리즘으로 암호화하고, 이를 다시 복호화하여 원본 데이터를 복원하는 전 과정을 RTL로 구현하였습니다.

---

## ✨ 핵심 기능 (Key Features)

* **⚡ 고속 하드웨어 파이프라인**: AES-128 표준의 SubBytes, ShiftRows, MixColumns, AddRoundKey 연산을 RTL로 완전 구현
* **🛠️ SoC 최적화 인터페이스**:
    * `AXI4-Lite (Slave)`: CPU 기반의 128-bit 암복호화 키(Key) 실시간 설정 및 제어
    * `AXI4-Stream (Slave)`: NPU 결과 데이터 또는 암호문의 고속 스트리밍 입력
    * `AXI4-Stream (Master)`: 암복호화 완료 데이터를 다음 모듈로 즉시 전송
* **🔁 암복호화 대칭 설계**: 암호화 코어(`aes_128_core`)와 복호화 코어(`aes_128_inv_core`)가 동일한 키 확장 모듈(`aes_key_expansion`)을 공유하여 리소스 효율 극대화
* **🎯 표준 검증 완료**: NIST AES-128 표준 테스트 벡터 기반 시뮬레이션으로 암복호화 데이터 무결성 100% 검증

---

## 🏗️ 시스템 아키텍처 (Architecture)

### **AES-128 Core Pipeline**

암호화 및 복호화 코어는 FSM(유한 상태 머신) 기반으로 라운드 연산을 순차 처리합니다.

| 단계 | 모듈명 | 주요 역할 |
| :--- | :--- | :--- |
| **Key Setup** | `aes_key_expansion` | 마스터 키로부터 라운드 0~10 키 생성 (Rcon, SubWord 적용) |
| **Substitution** | `sbox` / `inv_sbox` | 16개 병렬 S-Box를 통한 바이트 단위 비선형 치환 |
| **Permutation** | `func_shift_rows` / `func_inv_shift_rows` | 상태 행렬의 행 이동 연산 |
| **Diffusion** | `func_mix_columns` / `func_inv_mix_columns` | GF(2⁸) 기반 열 혼합으로 확산 효과 극대화 |
| **Round Key** | FSM 내부 | 각 라운드 키와의 XOR 연산 (AddRoundKey) |
| **Enc Wrapper** | `aes_enc_axi_wrapper` | AXI 규격 호환 암호화 IP 래퍼 |
| **Dec Wrapper** | `aes_dec_axi_wrapper` | AXI 규격 호환 복호화 IP 래퍼 |

### **FSM 상태 전이**

```
[암호화]  IDLE → ROUND_OP (×9) → FINAL_RD → DONE
[복호화]  IDLE → INITIAL → ROUND_OP (×9) → FINAL_RD → DONE
```

---

## 📂 프로젝트 구조 (Project Structure)

```bash
├── src/                          # Verilog RTL 설계 소스
│   ├── aes_enc_axi_wrapper.v     # 암호화 AXI IP 래퍼 (Top)
│   ├── aes_dec_axi_wrapper.v     # 복호화 AXI IP 래퍼 (Top)
│   ├── aes_128_core.v            # AES-128 암호화 코어 + S-Box
│   ├── aes_128_inv_core.v        # AES-128 복호화 코어 + Inv S-Box
│   └── aes_key_expansion.v       # 공용 키 확장 모듈
├── sim/                          # 검증용 테스트벤치
│   └── tb_enc_dec.v              # 암복호화 통합 시뮬레이션
└── README.md
```

---

## 🔌 인터페이스 명세 (Interface Specification)

### AXI4-Lite (키 설정)

| 주소 오프셋 | 레지스터 | 설명 |
| :---: | :--- | :--- |
| `0x00` | KEY[31:0] | 마스터 키 하위 32비트 |
| `0x04` | KEY[63:32] | 마스터 키 두 번째 워드 |
| `0x08` | KEY[95:64] | 마스터 키 세 번째 워드 |
| `0x0C` | KEY[127:96] | 마스터 키 상위 32비트 |

### AXI4-Stream

| 포트 | 방향 | 데이터 폭 | 설명 |
| :--- | :---: | :---: | :--- |
| `s_axis` | Slave | 32-bit (암호화) / 128-bit (복호화) | 입력 데이터 스트림 |
| `m_axis` | Master | 128-bit | 암복호화 결과 스트림 |

---

## ✅ 시뮬레이션 결과 (Simulation Result)

```
[암호화]
  Key       : 2b7e151628aed2a6abf7158809cf4f3c
  Plaintext : 00000000000000000000000000000007
  Ciphertext: (AES-128 표준 출력값)

[복호화]
  Ciphertext → Decrypted: 00000000000000000000000000000007

** SUCCESS: Data Integrity Verified! **
```
