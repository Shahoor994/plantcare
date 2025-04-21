#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

// Replace with your Wi-Fi credentials
const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";

ESP8266WebServer server(80);

// LED Pins
const int GREEN_LED = D1; // Healthy
const int RED_LED = D2;   // Stained

void setup() {
  Serial.begin(115200);

  pinMode(GREEN_LED, OUTPUT);
  pinMode(RED_LED, OUTPUT);
  digitalWrite(GREEN_LED, LOW);
  digitalWrite(RED_LED, LOW);

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("\nConnected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  // Endpoint for stained detection (unhealthy)
  server.on("/stained", []() {
    digitalWrite(GREEN_LED, LOW);
    digitalWrite(RED_LED, HIGH); // Red ON
    server.send(200, "text/plain", "Stained Detected: RED LED ON");
  });

  // Endpoint for healthy detection
  server.on("/healthy", []() {
    digitalWrite(RED_LED, LOW);
    digitalWrite(GREEN_LED, HIGH); // Green ON
    server.send(200, "text/plain", "Healthy Leaf: GREEN LED ON");
  });

  server.begin();
  Serial.println("Server started...");
}

void loop() {
  server.handleClient();
}
