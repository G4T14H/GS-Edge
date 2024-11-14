# Documentação do Projeto de Monitoramento com ESP32, Sensor Ultrassônico, PIR e MQTT

## Integrantes do Projeto
- Gustavo - RM557876
- Pedro Paulo - RM554880
- Lucca - RM554608

---

## Índice
1. [Pré-requisitos](#pré-requisitos)
2. [Passo a Passo](#passo-a-passo)
   - [Passo 1: Conexão Wi-Fi](#passo-1-conexão-wi-fi)
   - [Passo 2: Configuração do Sensor Ultrassônico](#passo-2-configuração-do-sensor-ultrassônico)
   - [Passo 3: Configuração do Sensor PIR e Ações com LED e Buzzer](#passo-3-configuração-do-sensor-pir-e-ações-com-led-e-buzzer)
   - [Passo 4: Configuração do MQTT](#passo-4-configuração-do-mqtt)
   - [Passo 5: Teste e Monitoramento](#passo-5-teste-e-monitoramento)
3. [Configuração do Código](#configuração-do-código)
4. [Exibição dos Dados no MyMQTT](#exibição-dos-dados-no-mymqtt)
5. [Considerações Finais](#considerações-finais)

---

## Pré-requisitos
- **ESP32**
- **Sensor Ultrassônico HC-SR04**
- **Sensor de Movimento PIR**
- **Buzzer**
- **LED Vermelho**
- **Wi-Fi Configurado**
- **Conta no MyMQTT ou aplicativo equivalente para monitoramento MQTT**

---

## Passo a Passo

### Passo 1: Conexão Wi-Fi
- Configure a conexão Wi-Fi com o nome da rede (SSID) e senha, editando as informações no código.
- O ESP32 se conectará ao Wi-Fi e indicará no monitor serial quando estiver conectado.

### Passo 2: Configuração do Sensor Ultrassônico
- Conecte o sensor HC-SR04 aos pinos de **Trigger** e **Echo** no ESP32.
- O código mede a distância do objeto à frente do sensor. Ele acende o LED se a distância estiver entre 250 e 350 cm e aciona o buzzer se estiver abaixo de 250 cm.

### Passo 3: Configuração do Sensor PIR e Ações com LED e Buzzer
- Conecte o sensor PIR ao pino de entrada digital do ESP32.
- Quando o sensor PIR detecta movimento, o monitoramento de proximidade inicia:
  - O LED acende para distâncias entre 250 e 350 cm.
  - Buzzer e LED são acionados quando a distância é menor que 250 cm.

### Passo 4: Configuração do MQTT
- Configure o cliente MQTT inserindo o ID do cliente, endereço do broker MQTT e tópicos de envio e recebimento.
- A função de callback do MQTT controla o LED remotamente.
- A cada detecção, o ESP32 envia mensagens MQTT com informações sobre objetos detectados.

### Passo 5: Teste e Monitoramento
- Abra o monitor serial para verificar os dados de distância.
- Inscreva-se nos tópicos MQTT no MyMQTT para monitorar as mensagens enviadas pelo ESP32.

---

## Considerações Finais
 Este projeto de monitoramento com ESP32, sensores ultrassônico e PIR, LED, buzzer e comunicação MQTT oferece uma solução prática para detecção de presença e monitoramento de distância em tempo real. Utilizando o aplicativo MyMQTT, os dados de detecção podem ser visualizados remotamente, proporcionando maior flexibilidade e controle em diversos cenários.

## Potenciais Melhorias
Integração com Aplicativos Web: Expandir a exibição dos dados para uma interface web para facilitar o monitoramento em desktops e dispositivos móveis.
Notificações Push: Enviar alertas para dispositivos móveis sempre que um objeto seja detectado ou quando o LED seja acionado.
Aprimoramento de Sensores: Adicionar sensores adicionais, como câmeras ou sensores de temperatura, para aumentar a funcionalidade do sistema.

## Limitações
A precisão dos sensores pode variar em função das condições ambientais (por exemplo, interferência do sinal Wi-Fi, objetos refletivos).
A utilização do MQTT depende de uma conexão de rede estável, o que pode limitar o uso em áreas sem cobertura de internet.
Este projeto é uma base para soluções mais complexas de automação e segurança residencial, podendo ser adaptado para diversas outras aplicações.

---
## Link do Wokwi
https://wokwi.com/projects/414552662883148801

![image](https://github.com/user-attachments/assets/82b5a1a7-373a-44e7-b485-9ce3c6d957fc)

---

## Configuração do Código

```python
from machine import Pin, PWM
import time
from time import sleep
from hcsr04 import HCSR04
import network
import ujson
from umqtt.simple import MQTTClient

# Configurações de pinos e sensores
sensor_de_distancia = HCSR04(trigger_pin=5, echo_pin=18)
medidor_de_distancia = sensor_de_distancia.distance_cm()
buzzer = PWM(Pin(12), freq=1000, duty=0)
pin_motion = Pin(25, Pin.IN)
led_red = Pin(26, Pin.OUT)

while True:
    if pin_motion.value() == 1:
        if 250 < medidor_de_distancia < 350:
            led_red.on()
        elif medidor_de_distancia < 250:
            led_red.on()
            buzzer.duty(1023)
            print("Objeto Detectado a:", medidor_de_distancia, "cm.")
    elif pin_motion.value() == 0:
        if 350 <= medidor_de_distancia < 400:
            buzzer.duty(0)
            led_red.off()
            print("Nenhum Objeto Detectado.")
    sleep(2)

# Parâmetros MQTT
MQTT_CLIENT_ID = "stein"
MQTT_BROKER = "broker.mqttdashboard.com"
MQTT_TOPIC1 = "exp.leds"
MQTT_TOPIC2 = "exp.led"

def callback(topic, msg):
    global status
    if str(msg.decode()) == "Ligado" and status == 0:
        client.publish(MQTT_TOPIC2, "Ligado")
        status = 1
        led_red.on()
    elif str(msg.decode()) == "Ligado" and status == 1:
        led_red.off()
        status = 0
        client.publish(MQTT_TOPIC2, "Desligado")

# Conexão com Wi-Fi e servidor MQTT
sta_if = network.WLAN(network.STA_IF)
sta_if.active(True)
sta_if.connect('Wokwi-GUEST', '')
while not sta_if.isconnected():
    time.sleep(0.1)

client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER)
client.set_callback(callback)
client.connect()
client.subscribe(MQTT_TOPIC1)

status = 0
while True:
    client.check_msg()
    time.sleep(1)

