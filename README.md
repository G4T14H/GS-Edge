# Documentação do Projeto de Monitoramento com ESP32, Sensor Ultrassônico, PIR e MQTT

## Integrantes do Projeto
- Gustavo - RM557876
- Pedro Paulo - RM554880
- Lucca - RM554608

---

## 📋 Descrição do Projeto
Este projeto utiliza um microcontrolador **ESP32**, sensores ultrassônicos (**HC-SR04**) e de movimento (**PIR**) para monitoramento de distância e detecção de presença, integrando comunicação **MQTT** para publicação e controle remoto. Quando movimentos ou objetos são detectados, o sistema responde com ações locais, como acionamento de LED e buzzer, além de enviar notificações ao broker MQTT. A visualização dos dados é feita através do aplicativo **MyMQTT** ou outro cliente MQTT.

---

## 🗂️ Índice
1. [Pré-requisitos](#pré-requisitos)
2. [Componentes Utilizados](#componentes-utilizados)
3. [Montagem do Circuito](#montagem-do-circuito)
4. [Instalação e Configuração](#instalação-e-configuração)
   - [Configuração Wi-Fi](#configuração-wi-fi)
   - [Configuração MQTT](#configuração-mqtt)
5. [Uso do Sistema](#uso-do-sistema)
6. [Código Completo](#código-completo)
7. [Arquivos Disponíveis](#arquivos-disponíveis)
8. [Considerações Finais](#considerações-finais)

---

## 🛠️ Pré-requisitos
Certifique-se de ter os seguintes itens antes de começar:
- **Placa ESP32**
- **Sensor Ultrassônico HC-SR04**
- **Sensor de Movimento PIR**
- **Buzzer**
- **LED Vermelho**
- **Resistores apropriados (para o LED)**
- **Acesso à internet para conexão Wi-Fi**
- **Cliente MQTT (como MyMQTT)**

---

## 🧰 Componentes Utilizados
- **ESP32:** Microcontrolador principal.
- **HC-SR04:** Sensor de distância ultrassônico.
- **PIR:** Sensor de movimento passivo.
- **Buzzer:** Emitir som para alertas.
- **LED:** Indicar estados de detecção.

---

## ⚡ Montagem do Circuito
Abaixo, a ligação de cada componente ao ESP32:

| Componente         | Pino ESP32    |
|---------------------|---------------|
| Trigger (HC-SR04)  | GPIO 5        |
| Echo (HC-SR04)     | GPIO 18       |
| PIR Sensor         | GPIO 25       |
| Buzzer             | GPIO 12       |
| LED Vermelho       | GPIO 2        |

**Esquema de conexão:**  
Use resistores de 220 ohms para proteger o LED. O HC-SR04 deve ser ligado com cuidado para evitar picos de corrente.

---

## link da montagem do dispositivo no wokwi 

https://wokwi.com/projects/414946990002981889
![image](https://github.com/user-attachments/assets/b6d55569-ab65-4d10-9cfe-a7a19f4c2a75)


## 🖥️ Instalação e Configuração
### 1. Configuração Wi-Fi
```cpp
const char* ssid = "SEU_SSID";
const char* password = "SUA_SENHA";
2. Configuração MQTT
Insira o broker MQTT e os tópicos desejados:

const char* mqtt_server = "broker.hivemq.com"; // Broker público
Os tópicos utilizados para publicação são:

motionTopic: Indica quando o movimento é detectado ou parado.
distanceTopic: Publica a distância medida pelo sensor ultrassônico.
controlTopic: Para controle remoto do LED.
3. Upload do Código
Conecte o ESP32 ao computador.
Instale o Arduino IDE ou VS Code com PlatformIO.
Adicione as bibliotecas necessárias:
WiFi.h
PubSubClient.h
Faça o upload do código na placa.
🚀 Uso do Sistema
Inicialização:
Após o upload, o ESP32 se conectará ao Wi-Fi e ao servidor MQTT.
No monitor serial, você verá as mensagens de status.
Detecção de Movimento:
Ao detectar movimento com o PIR, o LED acende e o buzzer emite som.
Medição de Distância:
O sensor ultrassônico calcula a distância em tempo real:
Entre 250-350 cm: Acende o LED.
Abaixo de 250 cm: Acende o LED e ativa o buzzer.
Controle MQTT:
Use o aplicativo MyMQTT para enviar mensagens ao tópico controlTopic:
Enviar "Ligado" aciona o LED.
Enviar "Desligado" apaga o LED.
💻 Código Completo
Abaixo está o código utilizado no projeto:

#include <WiFi.h>
#include <PubSubClient.h>

// Configurações da rede Wi-Fi
const char* ssid = "Wokwi-GUEST";
const char* password = "";

// Configuração do servidor MQTT
const char* mqtt_server = "broker.hivemq.com"; // Broker MQTT público

// Definição dos pinos
#define PIR_PIN 25 // Pino do sensor PIR
#define BUZZER_PIN 12 // Pino do buzzer
#define LED_PIN 2 // Pino do LED (resistor conectado)
#define TRIG_PIN 5 // Pino TRIG do sensor ultrassônico
#define ECHO_PIN 18 // Pino ECHO do sensor ultrassônico

// Variáveis para controle
WiFiClient espClient;
PubSubClient client(espClient);
char msg[50];
long duration;
int distance;
bool pirState = LOW;

// Função para configurar a conexão Wi-Fi
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando-se a ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWi-Fi conectado!");
  Serial.print("Endereço IP: ");
  Serial.println(WiFi.localIP());
}

// Callback para tratar mensagens recebidas do MQTT
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Mensagem recebida [");
  Serial.print(topic);
  Serial.print("]: ");
  
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

// Função para reconectar ao servidor MQTT
void reconnect() {
  while (!client.connected()) {
    Serial.print("Tentando conectar ao MQTT...");
    
    String clientId = "ESP32Client-";
    clientId += String(random(0xffff), HEX);
    
    if (client.connect(clientId.c_str())) {
      Serial.println("Conectado ao MQTT!");
      client.subscribe("controlTopic"); // Inscreve-se no tópico de controle
    } else {
      Serial.print("Falha na conexão, rc=");
      Serial.print(client.state());
      Serial.println(" Tentando novamente em 5 segundos...");
      delay(5000);
    }
  }
}

// Configurações iniciais
void setup() {
  pinMode(PIR_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  Serial.begin(115200); // Inicializa a comunicação serial
  setup_wifi();         // Configura a conexão Wi-Fi
  
  client.setServer(mqtt_server, 1883); // Configura o servidor MQTT
  client.setCallback(callback);       // Define o callback para mensagens recebidas
}

// Função para calcular a distância usando o sensor ultrassônico
int getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  duration = pulseIn(ECHO_PIN, HIGH);
  distance = duration * 0.034 / 2; // Converte o tempo para distância em cm
  return distance;
}

// Loop principal
void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Sensor PIR
  int pirVal = digitalRead(PIR_PIN);
  if (pirVal == HIGH && !pirState) {
    Serial.println("Movimento detectado!");
    client.publish("motionTopic", "Movimento detectado");
    digitalWrite(LED_PIN, HIGH);
    digitalWrite(BUZZER_PIN, HIGH);
    pirState = true;
  } else if (pirVal == LOW && pirState) {
    Serial.println("Movimento parado!");
    client.publish("motionTopic", "Movimento parado");
    digitalWrite(LED_PIN, LOW);
    digitalWrite(BUZZER_PIN, LOW);
    pirState = false;
  }

  // Sensor ultrassônico
  int distance = getDistance();
  Serial.print("Distância: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Publica a distância no MQTT
  snprintf(msg, 50, "Distância: %d cm", distance);
  client.publish("distanceTopic", msg);

  delay(1000); // Aguarda 1 segundo antes de repetir
}
