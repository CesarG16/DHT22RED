## CESAR AUGUSTO GONZALEZ INFANTE
## DHT22 CON NODE RED
## Material Necesario

Para realizar esta practica se usaran los siguientes elementos:

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Sensor DHT11
- [NODERED](http://localhost:1880/)



## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="CESARGONZALEZ";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("CESAR", output.c_str());
  }
}

```

2. Instalar la libreria de **DHT sensor library for ESPx** como se muestra en la siguente imagen.

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE1.png?raw=true)

3. Hacer la conexion de **DHT11** con la **ESP32** como se muestra en la siguente imagen.

![](https://github.com/CesarG16/DHT22/blob/main/eje3.png?raw=true)


### Instrucciónes de operación

1. Iniciar simulador.
2. Visualizar los datos en el monitor serial.
3. Colocar la temperatura y humedad dando *doble click* al sensor **DHT11** 
4. Abrir Simbolo de sistema como administrador tras instalar el programa node red y escribir  node-red en la terminal del sistema tras esto ir a un navegador web y escribir: (http://localhost:1880/) y estaremos en node red.

5. Juntar los elementos siguientes mostrados en la pantalla:

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE3.png?raw=true)

6. Los elementos puestos son : mqtt in, json, function, function, gauge, gauge y chart.
7. Después abrir Dashboard en la esquina superior de la derecha como se ve en la siguiente imagen:

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE4.png?raw=true)

8. Dar click en +tab.
9. Dar click en +group y añadir dos. 
10. De alli tanto en tab como en los grupos se les da en los botones de editar de cada uno y en la seccion de name se le asigna un nombre a cada uno.
11. de alli se da click a cada uno de los elementos unidos anteriormente y se editan empezando por el primero, hasta el ultimo como se ven en las siguientes imágenes:

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE5.png?raw=true)
![](https://github.com/CesarG16/DHT22RED/blob/main/EJE6.png?raw=true)
![](https://github.com/CesarG16/DHT22RED/blob/main/EJE7.png?raw=true)
![](https://github.com/CesarG16/DHT22RED/blob/main/EJE8.png?raw=true)
![](https://github.com/CesarG16/DHT22RED/blob/main/EJE9.png?raw=true)
![](https://github.com/CesarG16/DHT22RED/blob/main/EJE10.png?raw=true)
![](https://github.com/CesarG16/DHT22RED/blob/main/EJE11.png?raw=true)

12. Tras esto se da en el boton rojo de arriba a la derecha que dice Deploy y cuando esté listo se le da en el siguiente boton que esta abajo de Deploy para que abra una nueva página y se visualice el resultado:

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE11.png?raw=true)

## Resultados


Cuando haya funcionado, verás los valores dentro del monitor serial como se muestra en la siguente imagen y en la página creada por node - red.

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE2.png?raw=true)
![](https://github.com/CesarG16/DHT22RED/blob/main/EJE13.png?raw=true)



## Evidencias

[Página](https://wokwi.com/projects/367817377098693633)


# Créditos

Desarrollado por Ing. Cesar Augusto Gonzalez Infante

- [GitHub](https://github.com/CesarG16)