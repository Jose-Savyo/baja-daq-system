# ESP32 CVT Datalogger - Baja SAE

![Status: Concluído](https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen)
![License: MIT](https://img.shields.io/badge/license-MIT-blue)
![Language: C++](https://img.shields.io/badge/language-C++-orange)

Sistema de aquisição de dados (DAQ) de alta performance desenvolvido para a equipe **ParahyBaja (UFCG)**. O objetivo principal é monitorar a variação da relação de transmissão da CVT durante testes de aceleração em campo em protótipos Baja SAE.

## 📌 Visão Geral

O projeto consiste em um datalogger capaz de realizar amostragens em alta frequência (**3 kHz**) para capturar pulsos de sensores de rotação nos eixos primário e secundário da CVT. Os dados são armazenados em um buffer interno de 360KB e transmitidos via Wi-Fi (TCP/IP) para um computador após o ensaio.

### 🔌 Hardware Representativo
| ESP32 DevKit V1 | Sensor Indutivo (Exemplo) |
| :---: | :---: |
| <img src="http://googleusercontent.com/image_collection/image_retrieval/13179462552155900217_0" width="250px"> | <img src="http://googleusercontent.com/image_collection/image_retrieval/3643492005453445754_0" width="250px"> |

---

## 🚀 Funcionalidades Técnicas

* **Frequência de Amostragem:** 3 kHz estável (leitura a cada 333µs).
* **Capacidade de Armazenamento:** Buffer de 60.000 amostras na SRAM interna (aprox. 20 segundos de ensaio).
* **Arquitetura Dual-Core:**
    * **Core 0 (Comunicação):** Gerenciamento de rede (Access Point), Servidor TCP e controle de estados.
    * **Core 1 (Aquisição):** Loop crítico de tempo real dedicado apenas à coleta, garantindo **zero jitter** causado pelo Wi-Fi.
* **Transferência de Dados:** Protocolo TCP para garantir que nenhum pacote seja perdido durante o descarregamento do buffer.

## 📂 Estrutura do Repositório

* `/firmware`: Código fonte final e estável para o ESP32.
* `/scripts`: Scripts Python para recebimento e conversão de dados binários.
* `/data_samples`: Exemplos de logs coletados para testes de visualização.

## 🛠️ Como Utilizar

1.  **Hardware:** Conecte os sensores de rotação nos pinos configurados no firmware (utilize divisores de tensão se os sensores operarem em 12V).
2.  **Firmware:** Carregue o código da pasta `/firmware` no seu ESP32.
3.  **Conexão:** O ESP32 criará um ponto de acesso Wi-Fi chamado `ESP32-CVT-Logger`. Conecte seu computador a ele.
4.  **Coleta:** No seu terminal, execute o script cliente:
    ```bash
    python scripts/client_v1.py
    ```
5.  **Resultado:** O script enviará o sinal de "Start", aguardará 20s e baixará automaticamente o arquivo `.csv` com os dados.

## 📊 Estrutura de Dados (Packet)

Cada amostra é compactada em uma `struct` de 6 bytes para otimizar o uso da RAM:
```cpp
struct PontoDeColeta {
  byte estadoSensor1;         // 1 byte
  byte estadoSensor2;         // 1 byte
  unsigned long timestamp_us; // 4 bytes
};
