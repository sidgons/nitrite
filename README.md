//bibliotecas
#include <WiFi.h>                                                                 
#include <WiFiClient.h>
#include <Wire.h>
#include <SparkFun_APDS9960.h>
//#include "Adafruit_TCS34725.h"
#include "SSD1306.h"
#include <WiFiAP.h>

//variaveis 
float VALOR_R = 0;
float VALOR_G = 0;
float VALOR_B = 0;
//coordenadas display
#define POS_X_1     2
#define POS_Y_1     5
#define POS_X_2     2
#define POS_Y_2    20
#define POS_X_3     2
#define POS_Y_3    35
SparkFun_APDS9960 apds = SparkFun_APDS9960();
//Adafruit_TCS34725 tcs = Adafruit_TCS34725(TCS34725_INTEGRATIONTIME_2_4MS, TCS34725_GAIN_60X);
SSD1306  screen(0x3c, 5, 4);

WiFiServer server(80);
const char *ssid = "esp32";
const char *password = "123456789";
//codigo html
String recebe_dados;
String recebe_dados2;
String estado_do_pino_22 = "off";
const int porta_22 = 22;
String estado_do_pino_23 = "off";
const int porta_23 = 23;  
int leitura_sensor = 0;
const int porta_34 = 17;
uint16_t ambient_light = 0;
uint16_t red_light = 0;
uint16_t green_light = 0;
uint16_t blue_light = 0;
//funcoes
void setupDisplay() {
    screen.init();
    apds.init();
    //tcs.begin();
    screen.setFont(ArialMT_Plain_16);
}
int setupWifi(){
   Serial.begin(115200);
    WiFi.softAP(ssid, password);
    IPAddress myIP = WiFi.softAPIP();
    server.begin();
    
   return myIP;
   
}
void conectaWifi() {
    screen.clear();
    screen.drawString(POS_X_1, POS_Y_1, "Configurando ");
    screen.drawString(POS_X_2,POS_Y_2,"access point...");
    setupWifi();
    screen.display();
    delay(5000);
    screen.clear();
    screen.drawString(POS_X_1, POS_Y_1, "Server started");
    screen.drawString(POS_X_2,POS_Y_2,"IP address:");
    screen.drawString(POS_X_3,POS_Y_3,"192.168.4.1");
    screen.display();
}
void coletadados_dados_RGB(){
  uint16_t r, g, b;
  float R,G,B;
  //tcs.getRawData(&r, &g, &b, &c);
  //Pega os valores "crus" do sensor referentes ao Vermelho(r), Verde(g), Azul(b) e da Claridade(c)
  apds.readRedLight(r); 
  apds.readGreenLight(g); 
  apds.readBlueLight(b);
    //tcs.getRGB(&r, &g, &b);
  VALOR_R = r;
  VALOR_G = g;
  VALOR_B = b;
 
}
void displayPag2(){
    screen.clear();
    screen.drawString(POS_X_1, POS_Y_1, "Novo cliente ");
}
void displayPag3(){
    screen.clear();
    screen.drawString(POS_X_1, POS_Y_1, "Cliente desconectou ");
}

void comunicacao(){
     WiFiClient client = server.available();
     if (client) {
     
      String linha_atual = "";
      while (client.connected()){
        
        
        if (client.available()){
          char c = client.read();
          recebe_dados += c;
          if (c == '\n') {
            if (linha_atual.length() == 0) {
              client.println("HTTP/1.1 200 OK");
              client.println("Content-type:text/html");
              client.println("Connection: close");
              client.println();
              //displayPag2();
              if (recebe_dados.indexOf("GET /LED/on") >= 0) {
                estado_do_pino_22 = "on";
                digitalWrite(porta_22, HIGH);
              }
                else if (recebe_dados.indexOf("GET /LED/off") >= 0) {
                estado_do_pino_22 = "off";
                digitalWrite(porta_22, LOW);
                }
              if (recebe_dados.indexOf("GET /LED2/on") >= 0) {
                estado_do_pino_23 = "on";
                digitalWrite(porta_23, HIGH);
              }
                else if (recebe_dados.indexOf("GET /LED2/off") >= 0) {
                estado_do_pino_23 = "off";
                digitalWrite(porta_23, LOW);
                }
              if (recebe_dados.indexOf("GET /LER/DADOS") >= 0) {
                leitura_sensor=1;
                coletadados_dados_RGB();
                digitalWrite(porta_34, HIGH);
              }
               else if (recebe_dados.indexOf("GET /ZERAR") >= 0) {
                leitura_sensor=0;
                digitalWrite(porta_34, LOW);
                }
              
              client.println("<!DOCTYPE html><html>");
              client.println("<head><title>Data collect RGB</title><meta name=\"viewport\" content=\"width=device-width, initial-scale=1\"><meta charset=\"utf-8\" />"); 
              client.println("<link rel=\"icon\" href=\"data:,\">");
              client.println("<style>html { font-family: Helvetica; display: inline-block; margin: 0px auto; text-align: center;}");
              client.println(".button { background-color: #4CAF50; border: none; color: white; padding: 16px 40px;box-shadow:5px 5px 7px;");
              client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
              client.println(".button2 { background-color: #ff0000; border: none; color: white; padding: 16px 40px;box-shadow:5px 5px 7px;");
              client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
              client.println(".button3 { background-color: #3a3939; border: none; color: white; padding: 16px 40px;box-shadow:5px 5px 7px;");
              client.println("text-decoration: none; font-size: 30px; margin: 2px; cursor: pointer;}");
              client.println("</style></head>");
              client.println("<body><h1>Sensor data</h1><p>");
             
              client.println("<p>Door LED  22 - Status: " + estado_do_pino_22 + "</p>");
              if (estado_do_pino_22 == "off") {
                client.println("<p><a href=\"/LED/on\"><button class=\"button\">ON</button></a></p>");
              }
              else {
                client.println("<p><a href=\"/LED/off\"><button class=\"button button2\">OFF</button></a></p>");
              }
              client.println("<p>Door LED 23 - Status: " + estado_do_pino_23 + "</p>");
              
              if (estado_do_pino_23 == "off") {
                client.println("<p><a href=\"/LED2/on\"><button class=\"button\">ON</button></a></p>");
              }
              else {
                client.println("<p><a href=\"/LED2/off\"><button class=\"button button2\">OFF</button></a></p>");
              }
              
              if (leitura_sensor == 0) {
                client.println("<p><a href=\"/LER/DADOS\"><button class=\"button3\">Measure</button></a></p>");
              }
              else {
                client.println("<p><a href=\"/ZERAR\"><button class=\"button3\">Zero</button></a></p>");
                 VALOR_R = 0;
                 VALOR_G = 0;
                 VALOR_B = 0;
              }
              client.println("RED:"+ (String)VALOR_R + "</p>");
              client.println("<p>GREEN:"+ (String)VALOR_G + "</p>");
              client.println("<p>BLUE:"+ (String)VALOR_B + "</p>");              
              client.println("<span style=\"font-size:12px;color:#121ab2;margin-top:30%;\">By Matheus</span></body></html>");
              client.println();
              break;
            }
            else {
              linha_atual = "";
            }
          }
          else if (c != '\r') {
            linha_atual += c;
          }
        }
      }
      recebe_dados = "";
      recebe_dados2 = "";
      client.stop();
     } 
    // displayPag3();        
}
//corpo do programa
void setup(){
    //Serial.begin(115200);
    setupDisplay();
    conectaWifi();
    pinMode(porta_22, OUTPUT);
    digitalWrite(porta_22, LOW);
    pinMode(porta_23, OUTPUT);
    digitalWrite(porta_23, LOW);
    pinMode(porta_34, OUTPUT);
    digitalWrite(porta_34, LOW);
}
void loop(){
  comunicacao();  
} 
