# PR-CTICA-9

INTRODUCCIÓN

Este ejercicio consiste únicamente en hacer conexion a un servidor.

DESCRIPCIÓN

Tomaremos los datos de la practica anteiror y agregaremos un sensor ultrasonico 

INSTRUCCIONES

1- En Wokwi colocaremos el codigo siguiente: #include <ArduinoJson.h> #include <WiFi.h> #include <PubSubClient.h> #define BUILTIN_LED 2 #include "DHTesp.h" const int DHT_PIN = 15; const int Trigger = 4; //Pin digital 2 para el Trigger del sensor const int Echo = 2; //Pin digital 3 para el Echo del sensor DHTesp dhtSensor; // Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST"; const char* password = ""; const char* mqtt_server = "3.65.168.153"; String username_mqtt="ACH13"; String password_mqtt="12345678";

WiFiClient espClient; PubSubClient client(espClient); unsigned long lastMsg = 0; #define MSG_BUFFER_SIZE (50) char msg[MSG_BUFFER_SIZE]; int value = 0;

void setup_wifi() {

delay(10); // We start by connecting to a WiFi network Serial.println(); Serial.print("Connecting to "); Serial.println(ssid);

WiFi.mode(WIFI_STA); WiFi.begin(ssid, password);

while (WiFi.status() != WL_CONNECTED) { delay(500); Serial.print("."); }

randomSeed(micros());

Serial.println(""); Serial.println("WiFi connected"); Serial.println("IP address: "); Serial.println(WiFi.localIP()); }

void callback(char* topic, byte* payload, unsigned int length) { Serial.print("Message arrived ["); Serial.print(topic); Serial.print("] "); for (int i = 0; i < length; i++) { Serial.print((char)payload[i]); } Serial.println();

// Switch on the LED if an 1 was received as first character if ((char)payload[0] == '1') { digitalWrite(BUILTIN_LED, LOW);
// Turn the LED on (Note that LOW is the voltage level // but actually the LED is on; this is because // it is active low on the ESP-01) } else { digitalWrite(BUILTIN_LED, HIGH);
// Turn the LED off by making the voltage HIGH }

}

void reconnect() { // Loop until we're reconnected while (!client.connected()) { Serial.print("Attempting MQTT connection..."); // Create a random client ID String clientId = "ESP8266Client-"; clientId += String(random(0xffff), HEX); // Attempt to connect if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) { Serial.println("connected"); // Once connected, publish an announcement... client.publish("outTopic", "hello world"); // ... and resubscribe client.subscribe("inTopic"); } else { Serial.print("failed, rc="); Serial.print(client.state()); Serial.println(" try again in 5 seconds"); // Wait 5 seconds before retrying delay(5000); } } }

void setup() { pinMode(BUILTIN_LED, OUTPUT); // Initialize the BUILTIN_LED pin as an output Serial.begin(115200); setup_wifi(); client.setServer(mqtt_server, 1883); client.setCallback(callback); dhtSensor.setup(DHT_PIN, DHTesp::DHT22); pinMode(Trigger, OUTPUT); //pin como salida pinMode(Echo, INPUT); //pin como entrada digitalWrite(Trigger, LOW);//Inicializamos el pin con 0 }

void loop() {

delay(1000); TempAndHumidity data = dhtSensor.getTempAndHumidity(); long t; //timepo que demora en llegar el eco long d; //distancia en centimetros

digitalWrite(Trigger, HIGH); delayMicroseconds(10); //Enviamos un pulso de 10us digitalWrite(Trigger, LOW);

t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso d = t/59; //escalamos el tiempo a una distancia en cm

Serial.print("Distancia: "); Serial.print(d); //Enviamos serialmente el valor de la distancia Serial.print("cm"); Serial.println(); if (!client.connected()) { reconnect(); } client.loop();

unsigned long now = millis(); if (now - lastMsg > 2000) { lastMsg = now; //++value; //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

StaticJsonDocument<128> doc;

doc["DEVICE"] = "ESP32";
//doc["Anho"] = 2022;
doc["DISTANCIA"] = String(d);
doc["TEMPERATURA"] = String(data.temperature, 1);
doc["HUMEDAD"] = String(data.humidity, 1);


String output;

serializeJson(doc, output);

Serial.print("Publish message: ");
Serial.println(output);
Serial.println(output.c_str());
client.publish("Cisneros1", output.c_str());

2- Instalamos las librerias que utilizaremos

![Captura de pantalla 2024-01-27 005844](https://github.com/robertopatino42/PR-CTICA-9/assets/153964688/47dfdb8c-0d12-4af7-af11-ebc4a6ea82b5)

3- Agregar el Sensor DHT22 y realizar las conexiones con la ESP22


![Captura de pantalla 2024-01-27 005844](https://github.com/robertopatino42/PR-CTICA-9/assets/153964688/ba4e8d39-cbd1-4909-b8b4-3ccec38262b6)

4- En el programa Node Red vamos a instalar el bloque de mqtt in, posteriormente bloque json , así mismo las funciones y bloques de Chart y Gaugue Correspondientes.

-En bloque mqtt in colocamos el nombre del tópico y el numero de la IP que se utilizara (esta dede de ser la misma que la que se coloco en el codigo) -El bloque json colocar la accion de Always convert to JavaScript Object -En los bloques function colocaremos estos codigos uno en cada uno ;

msg.payload = msg.payload.TEMPERATURA;
msg.topic = "TEMPERATURA";
return msg;

msg.payload = msg.payload.HUMEDAD;
msg.topic = "HUMEDAD";
return msg;

msg.payload = msg.payload.DISTANCIA;
msg.topic = "DISTANCIA";
return msg;
-En bloques de Chart y Gaugue Selecionar los grupos de chart en graficos y los gaugue en indicador, colocar los datos que corresponden.

![Captura de pantalla 2024-01-20 094746](https://github.com/robertopatino42/PR-CTICA-9/assets/153964688/ea9bb1dc-ba46-4b19-b33e-18809490b55e)

![Captura de pantalla 2024-01-27 010721](https://github.com/robertopatino42/PR-CTICA-9/assets/153964688/23c29da9-d229-4096-ad11-b40080f814ea)


