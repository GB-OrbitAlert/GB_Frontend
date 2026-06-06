# OrbitAlert 🛰️
### Sistema de Alertas de Emergência via Satélite
**Global Solution 2026 — FIAP Engenharia de Software**

## Sobre o projeto

O OrbitAlert monitora enchentes e queimadas em tempo real combinando sensores físicos no campo com dados satelitais da NASA e do INPE. Utiliza o módulo **LoRa SX1276 na frequência 915 MHz** — banda ISM homologada pela Anatel — para comunicação sem fio de longo alcance entre a unidade sensora no campo e a unidade receptora na base comunitária, sem necessidade de internet ou celular.

---

---

## Arquitetura do sistema

```
CAMPO (Transmissor)                      BASE (Receptor)
ESP32 DevKit V1                          ESP32 DevKit V1
  ├── DHT22 — temperatura e umidade        ├── ILI9341 2.8" TFT 240x320
  ├── HC-SR04 — distância ultrassônica     ├── Buzzer
  ├── Sensor nível d'água (analógico)      └── LoRa SX1276 (RX)
  └── LoRa SX1276 (TX)
           │                                      │
           └──────── 915 MHz LoRa ───────────────┘
                   Até ~3 km em campo aberto
                   SF10 | 125 kHz | CR 4/5
```

---

## Níveis de alerta

| Nível | Sensor nível (0–4095) | HC-SR04 | DHT22 | Ação |
|---|---|---|---|---|
| 🟢 **NORMAL** | < 1300 | > 180 cm | — | Monitoramento |
| 🟡 **ATENÇÃO** | > 1300 | < 180 cm | — | Display amarelo |
| 🟠 **CRÍTICO** | > 2600 | < 100 cm | — | Bip a cada 2 s |
| 🔴 **EMERGÊNCIA** | > 3700 | < 50 cm | T > 38°C + U < 20% | Buzzer contínuo + flash |

---

## Hardware necessário

### Transmissor (Campo)

| Componente | Quantidade |
|---|---|
| ESP32 DevKit V1 | 1 |
| Módulo LoRa SX1276 915 MHz (Ra-02 ou similar) | 1 |
| Sensor DHT22 | 1 |
| Sensor HC-SR04 | 1 |
| Sensor nível d'água (analógico) | 1 |
| Resistor 1kΩ + Resistor 2kΩ (divisor ECHO) | 1 de cada |
| LED de status (opcional) | 1 |
| Resistor 330Ω (LED) | 1 |

### Receptor (Base)

| Componente | Quantidade |
|---|---|
| ESP32 DevKit V1 | 1 |
| Módulo LoRa SX1276 915 MHz (Ra-02 ou similar) | 1 |
| Display TFT ILI9341 2.8" 240×320 | 1 |
| Buzzer passivo | 1 |

---

## Pinagem

### Transmissor — ESP32

| Componente | Pino ESP32 |
|---|---|
| LoRa SCK | GPIO 18 (SPI) |
| LoRa MISO | GPIO 19 (SPI) |
| LoRa MOSI | GPIO 23 (SPI) |
| LoRa CS (NSS) | GPIO 5 |
| LoRa RST | GPIO 14 |
| LoRa DIO0 | GPIO 2 |
| DHT22 DATA | GPIO 4 |
| HC-SR04 TRIG | GPIO 15 |
| HC-SR04 ECHO | GPIO 13 (*) |
| Sensor nível SIG | GPIO 34 |
| LED de status | GPIO 25 |

> (*) ECHO do HC-SR04 opera em 5V. Use divisor de tensão:
> `ECHO → R1(1kΩ) → GPIO 13` e `R2(2kΩ) → GND`

### Receptor — ESP32

| Componente | Pino ESP32 |
|---|---|
| LoRa SCK | GPIO 18 (SPI) |
| LoRa MISO | GPIO 19 (SPI) |
| LoRa MOSI | GPIO 23 (SPI) |
| LoRa CS (NSS) | GPIO 5 |
| LoRa RST | GPIO 14 |
| LoRa DIO0 | GPIO 2 |
| ILI9341 CS | GPIO 33 |
| ILI9341 DC | GPIO 17 |
| ILI9341 RST | GPIO 13 |
| ILI9341 BLK | 3.3V |
| Buzzer | GPIO 32 |

> LoRa e ILI9341 compartilham o barramento SPI (18/19/23).
> Cada módulo tem seu próprio CS — sem conflito.

---

## Parâmetros LoRa

```
Frequência  : 915 MHz  (banda ISM Brasil — Anatel)
SF          : 10       (Spreading Factor — alcance x velocidade)
Bandwidth   : 125 kHz  (padrão LoRaWAN)
Coding Rate : 4/5      (redundância mínima)
TX Power    : 20 dBm   (máximo do SX1276)
```

> Todos os parâmetros devem ser idênticos no transmissor e no receptor.

---

## Formato do pacote

O transmissor envia um pacote CSV pela rede LoRa:

```
NIVEL_AGUA,DISTANCIA_CM,TEMPERATURA,UMIDADE,ALERTA
```

Exemplo:
```
2800,45,40.2,17.3,EMERGENCIA:ENCHENTE
```

O receptor parseia o CSV e atualiza o display e o buzzer.

---

## Estrutura do repositório

```
orbitalert-lora/
├── transmissor/
│   └── transmissor.ino    # ESP32 + LoRa SX1276 — sensores e TX
├── receptor/
│   └── receptor.ino       # ESP32 + LoRa SX1276 + ILI9341 — RX
├── orbitalert_python/
│   ├── main.py            # Menu principal (4 funções + monitor serial)
│   ├── apis.py            # NASA FIRMS, NASA EONET, Open-Meteo, INPE
│   ├── risco.py           # Modelo exponencial R(x) = R₀ · eᵏˣ
│   ├── historico.py       # Histórico em JSON
│   ├── relatorio.py       # Relatórios com gráficos matplotlib
│   ├── serial_receptor.py # Leitura USB do receptor (pyserial)
│   └── requirements.txt
├── .gitignore
└── README.md
```

---

## Bibliotecas Arduino

### Transmissor
```
LoRa                  by Sandeep Mistry
DHT sensor library    by Adafruit
Adafruit Unified Sensor by Adafruit
```

### Receptor
```
LoRa                  by Sandeep Mistry
Adafruit ILI9341      by Adafruit
Adafruit GFX Library  by Adafruit
Adafruit BusIO        by Adafruit
```

Instale pelo **Sketch → Include Library → Manage Libraries** no Arduino IDE.

### platformio.ini

```ini
[env:esp32dev]
platform  = espressif32
board     = esp32dev
framework = arduino

; Transmissor
lib_deps =
    sandeepmistry/LoRa
    adafruit/DHT sensor library
    adafruit/Adafruit Unified Sensor

; Receptor
lib_deps =
    sandeepmistry/LoRa
    adafruit/Adafruit ILI9341
    adafruit/Adafruit GFX Library
    adafruit/Adafruit BusIO
```

---

## Software Python

```bash
git clone https://github.com/seu-usuario/orbitalert-lora.git
cd orbitalert-lora/orbitalert_python
pip install -r requirements.txt
python main.py
```

### Dependências

```
requests>=2.31.0    # APIs NASA, INPE, Open-Meteo
matplotlib>=3.7.0   # Gráficos e relatórios
pyserial>=3.5       # Monitor USB do receptor
```

### Chave NASA FIRMS

Obtenha gratuitamente em:
```
https://firms.modaps.eosdis.nasa.gov/api/map_key/
```

Crie o arquivo `config.py` (já no `.gitignore`):
```python
NASA_FIRMS_KEY = "sua_chave_aqui"
```

---

## Display ILI9341 2.8" — Layout da interface

```
┌─────────────────────────┐  y=0
│  OrbitAlert             │  Header (50px)
│  Global Solution · FIAP │
├─────────────────────────┤  y=50
│                         │
│     STATUS BANNER       │  Banner colorido (115px)
│    EMERGENCIA           │  Verde / Amarelo / Laranja / Vermelho
│    [ ENCHENTE ]         │  Pisca na emergência (500ms)
│                         │
├──────────────┬──────────┤  y=165
│ TEMPERATURA  │ UMIDADE  │  Grid de sensores (115px)
│  40.2 °C    │  17.3 % │
├──────────────┼──────────┤
│ NIVEL AGUA   │ DISTANCIA│
│    2800      │  45 cm  │
├─────────────────────────┤  y=280
│         RODAPÉ          │  (40px)
└─────────────────────────┘  y=320
```

---

## Modelo matemático (Python)

```
R(x) = R₀ × e^(k × x)

R₀ = risco base (temperatura × umidade inversa)
k  = 0.05 para queimada | 0.08 para enchente
x  = focos NASA FIRMS   | eventos NASA EONET
R  = probabilidade 0–100%
```

---

## Integração hardware + software

O receptor ESP32 envia dados via USB (Serial) no formato:

```
DATA:agua,distancia,temperatura,umidade,alerta,tipo
```

O Python lê em tempo real via `serial_receptor.py`. Use a opção `[5]` do menu para monitorar o campo ao vivo no terminal.

---

## APIs utilizadas

| API | O que fornece | Chave |
|---|---|---|
| NASA FIRMS | Focos de incêndio (VIIRS SNPP) | ✅ gratuita |
| NASA EONET | Eventos naturais ativos | ❌ |
| Open-Meteo | Temperatura, umidade, precipitação | ❌ |
| INPE Queimadas | Focos por estado no Brasil | ❌ |

---

---

## Impacto

| Público | Estimativa |
|---|---|
| Comunidades amazônicas sem cobertura móvel | ~8 milhões |
| Moradores de áreas de risco sem alerta offline | ~5,7 milhões |
| Comunidades indígenas e quilombolas | ~500 mil |
| **Total** | **~14 milhões de pessoas** |

Custo aproximado por kit: **R$ 337** — sem mensalidade, sem internet, sem celular.

---

## ODS atendidos

- **ODS 10** — Redução das desigualdades
- **ODS 11** — Cidades e comunidades sustentáveis
- **ODS 13** — Ação climática

---

## Equipe

| NOME | RM |
|---|---|
| GIANLUCA ANTONICCI | 570081
| ENZO VIEIRA PROVENZANO | 569696 |
| JOÃO VITOR RODRIGUEES COSTA | 569510 |

