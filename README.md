# 💡 Smart Lamp — ESP32 + FIWARE

Lâmpada inteligente com **ESP32**, **sensor de luminosidade** e **LED**, integrada à plataforma **FIWARE**, hospedada em uma instância **AWS EC2**, via **MQTT**. Pelo **Postman** é possível ligar e desligar a lâmpada e acompanhar, em tempo real, o estado dela e o nível de luminosidade do ambiente.

Projeto desenvolvido pelo grupo **PentaCode** (turma **1ESPH**) para o **CP4 de Edge Computing** — FIAP.

🎥 **Vídeo do projeto:** https://youtu.be/WTqTw4sdZzI

🔗 **Simulação no Wokwi:** https://wokwi.com/projects/475177956635328513

---

## 📋 Funcionalidades

- Ligar/desligar o LED remotamente por comandos enviados ao FIWARE
- Publicação contínua do estado da lâmpada (`on` / `off`)
- Leitura do sensor de luminosidade e envio do valor (0–100%)
- Consulta do valor atual e do histórico de luminosidade pelo Postman

## 🏗️ Arquitetura

```
                              ┌──────────── AWS EC2 (IP público) ────────────┐
┌──────────┐   MQTT :1883     │  ┌────────────┐        ┌──────────────────┐  │
│  ESP32   │ ────────────────►│  │ Mosquitto  │ ◄────► │ IoT Agent MQTT   │  │
│ (Wokwi)  │ ◄── comandos ────│  │  (broker)  │        │      :4041       │  │
│ LED+LDR  │                  │  └────────────┘        └────────┬─────────┘  │
└──────────┘                  │                                 │ NGSI       │
                              │  ┌────────────┐        ┌────────▼─────────┐  │
┌──────────┐   HTTP           │  │ STH-Comet  │ ◄───── │  Orion (CB)      │  │
│ Postman  │ ────────────────►│  │   :8666    │        │     :1026        │  │
└──────────┘                  │  └────────────┘        └──────────────────┘  │
                              └──────────────────────────────────────────────┘
```

O ESP32 (no Wokwi) e o Postman se comunicam com o FIWARE pelo **IP público da EC2**: o Wokwi usa esse IP como broker MQTT e o Postman como endereço das requisições HTTP.

## ☁️ Infraestrutura na AWS

Os componentes do FIWARE rodam em containers Docker dentro de uma instância **EC2** (Ubuntu). Para que o Wokwi e o Postman consigam acessar os serviços, o **Security Group** da instância precisa liberar estas portas de entrada:

| Porta | Protocolo | Serviço |
|---|---|---|
| 22 | TCP | SSH (acesso à VM) |
| 1883 | TCP | Mosquitto (broker MQTT) |
| 4041 | TCP | IoT Agent MQTT |
| 1026 | TCP | Orion Context Broker |
| 8666 | TCP | STH-Comet |

> ⚠️ Ao parar e iniciar a instância, o IP público da EC2 muda (a não ser que seja usado um **Elastic IP**). Nesse caso, atualize o IP no `sketch.ino` (`default_BROKER_MQTT`) e na variável `url` do Postman.

## 🔌 Hardware

| Componente | Função |
|---|---|
| ESP32 | Microcontrolador com Wi-Fi |
| LED | Lâmpada (liga/desliga) |
| Sensor de luminosidade | Leitura de luminosidade |

## 📡 Tópicos MQTT

| Tópico | Direção | Payload | Descrição |
|---|---|---|---|
| `/TEF/lamp001/cmd` | FIWARE → ESP32 | `lamp001@on\|` / `lamp001@off\|` | Comandos |
| `/TEF/lamp001/attrs` | ESP32 → FIWARE | `s\|on` / `s\|off` | Estado da lâmpada |
| `/TEF/lamp001/attrs/l` | ESP32 → FIWARE | `0` a `100` | Luminosidade (%) |

## 🚀 Como executar

1. Inicie a instância EC2 e confirme que os containers do FIWARE estão rodando (`sudo docker ps`).
2. Copie o **IP público** da instância no console da AWS.
3. Abra a simulação no [Wokwi](https://wokwi.com/projects/475177956635328513) e coloque esse IP em `default_BROKER_MQTT` no `sketch.ino`.
4. Inicie a simulação e aguarde a mensagem `Conectado com sucesso ao broker MQTT!` no monitor serial.
5. No Postman, use o IP público da EC2 no lugar de `IP_EC2` e envie os headers `fiware-service: smart` e `fiware-servicepath: /` nas requisições abaixo.

### Requisições no Postman

| Ação | Método | Endpoint |
|---|---|---|
| Verificar IoT Agent | GET | `http://IP_EC2:4041/iot/about` |
| Verificar Orion | GET | `http://IP_EC2:1026/version` |
| Ligar lâmpada | PATCH | `http://IP_EC2:1026/v2/entities/urn:ngsi-ld:Lamp:001/attrs` |
| Desligar lâmpada | PATCH | `http://IP_EC2:1026/v2/entities/urn:ngsi-ld:Lamp:001/attrs` |
| Estado da lâmpada | GET | `http://IP_EC2:1026/v2/entities/urn:ngsi-ld:Lamp:001/attrs/state` |
| Luminosidade atual | GET | `http://IP_EC2:1026/v2/entities/urn:ngsi-ld:Lamp:001/attrs/luminosity` |
| Histórico de luminosidade | GET | `http://IP_EC2:8666/STH/v1/contextEntities/type/Lamp/id/urn:ngsi-ld:Lamp:001/attributes/luminosity?lastN=30` |

Body para **ligar** a lâmpada:

```json
{ "on": { "type": "command", "value": "" } }
```

Body para **desligar** a lâmpada:

```json
{ "off": { "type": "command", "value": "" } }
```

## 📁 Estrutura

```
smart-lamp-esp32/
├── sketch.ino    # Código do ESP32
└── README.md
```

## 🛠️ Tecnologias

ESP32 · Arduino (C++) · PubSubClient · MQTT · AWS EC2 · Docker · FIWARE (Orion Context Broker, IoT Agent MQTT, STH-Comet) · Wokwi · Postman

## 👥 Integrantes — Grupo PentaCode · Turma 1ESPH

| Nome | RM |
|---|---|
| Felipe Garcia | RM 571741 |
| Andre Luiz | RM 573575 |
| Guilherme Amorim | RM 569024 |
| Arthur Jircik | RM 570754 |
| Matheus Marcelino | RM 571472 |
