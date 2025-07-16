#include <WiFi.h>              // Biblioteca para conexão WiFi
#include <HTTPClient.h>        // Biblioteca para fazer requisições HTTP
#include <UrlEncode.h>         // Biblioteca para codificar a URL
#include <DHT.h>               // Biblioteca para o sensor de temperatura e umidade

#define DHTPIN 15             // Pino onde o sensor DHT está conectado
#define DHTTYPE DHT11          // Tipo de sensor DHT (DHT11 ou DHT22)

const char* ssid = "Nome_da_minha_rede";        // Nome da rede WiFi
const char* password = "senha_da_rede"; // Senha do WiFi

DHT dht(DHTPIN, DHTTYPE);     // Instancia o sensor DHT

// Dados da API CallMeBot
String phoneNumber = "+55DDDmeunumero";  // Número do WhatsApp com código do país
String apiKey = "minha_chave";              // Chave de API obtida no CallMeBot

bool flag = 0;  // Flag de controle para evitar mensagens repetidas

// Função para enviar mensagens via CallMeBot
void sendMessage(String message) {
  if (WiFi.status() == WL_CONNECTED) {
    String url = "https://api.callmebot.com/whatsapp.php?phone=" + phoneNumber +
                 "&apikey=" + apiKey + "&text=" + urlEncode(message);

    HTTPClient http;
    http.begin(url);
    int httpResponseCode = http.GET();
    
    if (httpResponseCode == 200) {
      Serial.println("Mensagem enviada com sucesso!");
    } else {
      Serial.println("Erro no envio da mensagem");
      Serial.print("Código HTTP: ");
      Serial.println(httpResponseCode);
    }

    http.end();  // Libera recursos
  } else {
    Serial.println("ESP32 não está conectado ao WiFi");
  }
}

void setup() {
  Serial.begin(115200);   // Inicializa a porta serial para debug
  dht.begin();            // Inicializa o sensor DHT

  // Conecta ao WiFi
  WiFi.begin(ssid, password);
  Serial.print("Conectando ao WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.print("Conectado ao WiFi. IP: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  float temp = dht.readTemperature();  // Lê a temperatura

  if (isnan(temp)) {
    Serial.println("Falha ao ler a temperatura.");
  } else {
    Serial.print("Temperatura atual:");
    Serial.print(temp);
    Serial.println(" °C");

    if (temp >= 30 && flag == 0) {
      sendMessage("Alerta: Temperatura excedeu 30°C!");
      flag = 1;  // Atualiza flag para indicar que já foi enviado
    } 
    else if (temp < 30 && flag == 1) {
      sendMessage("Temperatura voltou ao normal (abaixo de 30°C).");
      flag = 0;  // Atualiza flag para indicar retorno à temperatura segura
    }
  }

  delay(10000);  // Espera 10 segundos antes de repetir
}
