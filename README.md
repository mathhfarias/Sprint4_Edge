# 🚑 Sistema Inteligente de Gestão de Filas para o Hospital das Clínicas

## 📋 Visão Geral do Projeto

Bem-vindo ao Sistema Inteligente de Gestão de Filas para o Hospital das Clínicas! Este projeto visa revolucionar a maneira como gerenciamos e monitoramos as filas fora do hospital. Utilizando o sensor ultrassônico HC-SR04, podemos medir com precisão o comprimento das filas, fornecendo dados valiosos para controlar a quantidade de pessoas que entram no hospital e melhorar a eficiência geral.

## 🎯 Objetivos

- **Monitorar o Comprimento das Filas**: Medir em tempo real o comprimento das filas fora do hospital.
- **Coleta de Dados**: Coletar e analisar dados para entender os horários de pico e o tempo médio de espera.
- **Melhorar a Eficiência**: Utilizar os dados coletados para otimizar o fluxo de pessoas entrando no hospital.

## 🛠️ Funcionalidades

- **Medição em Tempo Real das Filas**: Utilizando o sensor HC-SR04 para obter medições precisas.
- **Comunicação via MQTT**: Transmitir dados sem fio para um servidor central usando MQTT.
- **Visualização de Dados**: Exibir os dados coletados em um painel para fácil monitoramento e análise.

## 🚀 Começando

### Pré-requisitos

- Arduino Uno
- Sensor Ultrassônico HC-SR04
- Conta no TagoIO para visualização de dados
- Broker MQTT (por exemplo, Mosquitto)

### Configuração de Hardware

1. **Conexões**:
   - **HC-SR04 para Arduino**:
     - VCC ao 5V
     - GND ao GND
     - Trig ao Pino 4
     - Echo ao Pino 5

### Configuração de Software

1. **Arduino IDE**:
   - Instale as bibliotecas `WiFi` e `PubSubClient`.

2. **Código do Arduino**:
   ```cpp
   #include <WiFiEsp.h>
   #include <PubSubClient.h>

   // Pinos do sensor HC-SR04
   const int trigPin = 4;
   const int echoPin = 5;

   // Credenciais WiFi
   const char* ssid = "SEU_SSID";
   const char* password = "SUA_SENHA";

   // Configuração do broker MQTT
   const char* mqtt_server = "IP_DO_SEU_BROKER";
   WiFiEspClient espClient;
   PubSubClient client(espClient);

   void setup() {
     // Inicialização da comunicação serial
     Serial.begin(9600);
     esp8266.begin(115200); // Velocidade padrão de comunicação do ESP8266
     
     // Inicialização da biblioteca WiFiEsp
     WiFi.init(&esp8266);
     
     // Conectar ao WiFi
     connectWiFi();
     
     // Configurar o broker MQTT
     client.setServer(mqtt_server, 1883);
     
     // Configurar pinos do sensor
     pinMode(trigPin, OUTPUT);
     pinMode(echoPin, INPUT);
   }

   void loop() {
     if (!client.connected()) {
       reconnectMQTT();
     }
     client.loop();
     
     // Medir a distância
     long duration;
     int distance;
     
     digitalWrite(trigPin, LOW);
     delayMicroseconds(2);
     digitalWrite(trigPin, HIGH);
     delayMicroseconds(10);
     digitalWrite(trigPin, LOW);
     
     duration = pulseIn(echoPin, HIGH);
     distance = duration * 0.034 / 2;
     
     // Publicar a distância no broker MQTT
     char msg[50];
     snprintf(msg, 50, "Distancia: %d cm", distance);
     client.publish("sensor/distancia", msg);
     
     // Pausar por 1 segundo antes da próxima leitura
     delay(1000);
   }

   void connectWiFi() {
     Serial.print("Conectando ao WiFi...");
     while (WiFi.status() != WL_CONNECTED) {
       Serial.print(".");
       WiFi.begin(ssid, password);
       delay(5000);
     }
     Serial.println("conectado!");
   }

   void reconnectMQTT() {
     while (!client.connected()) {
       Serial.print("Tentando conectar ao broker MQTT...");
       if (client.connect("ArduinoClient")) {
         Serial.println("conectado");
       } else {
         Serial.print("falhou, rc=");
         Serial.print(client.state());
         Serial.println(" tentando novamente em 5 segundos");
         delay(5000);
       }
     }
   }
   ```

3. **Servidor MQTT (Node.js)**:
   ```javascript
   const mqtt = require('mqtt');
   const client = mqtt.connect('mqtt://IP_DO_SEU_BROKER');

   client.on('connect', () => {
     console.log('Conectado ao broker MQTT');
     client.subscribe('sensor/distancia', (err) => {
       if (!err) {
         console.log('Inscrito no tópico sensor/distancia');
       }
     });
   });

   client.on('message', (topic, message) => {
     console.log(`Tópico: ${topic}, Mensagem: ${message.toString()}`);
     // Processar a mensagem conforme necessário
   });
   ```

### Dashboard TagoIO

1. **Configurar Dispositivo**:
   - Crie um dispositivo no TagoIO para receber os dados do sensor.

2. **Configurar Widgets**:
   - Adicione widgets para visualizar os dados de distância em tempo real.
   - Configure esquemas de cores para diferentes intervalos de valores.

👥 **Colaboradores Sob Solution**
   - Matheus Farias de Lima - RM554254
   - Miguel Mauricio Parrado Patarroyo - RM554007
   - Vitor Pinheiro Nascimento - RM553693
   - Pedro Henrique Chaves - RM553988   

## 🤝 Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues e enviar pull requests.

## 📜 Licença

Este projeto está licenciado sob a [Licença SOB SOLUTION](LICENSE).
