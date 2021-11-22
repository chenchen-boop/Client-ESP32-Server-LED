# Client-ESP32-Server-LED
#include "WiFi.h"
#include "WebServer.h"

// SSID & Password
const char* ssid = "chenchen";
const char* password = "albert12345";

WebServer server(80);  // Object of WebServer(HTTP port, 80 is defult)

void setup() {
  pinMode(13,OUTPUT);
  pinMode(12,OUTPUT);
  ledcSetup(0, 5000, 10);
  ledcAttachPin(12, 0);


  
  Serial.begin(115200);
  Serial.println("Try Connecting to ");
  Serial.println(ssid);

  // Connect to your wi-fi modem
  WiFi.begin(ssid, password);

  // Check wi-fi is connected to wi-fi network
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(1000);
  }
  Serial.println("");
  Serial.println("WiFi connected successfully");
  Serial.print("Got IP: ");
  Serial.println(WiFi.localIP());  //Show ESP32 IP on serial

  //建立server的路徑
  createWebServer();
  
  //server啟動
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();  //handleClient要寫在loop
}

//----設定HTML資料----
String indexContent = "\
  <!DOCTYPE html>\
   <html>\
    <head>\
      <style>\
        body{ display: flex; flex-direction: column;}\
        a { background: #CEF2F4; width: 200px; height: 200px; margin: 20px; font-size: 30pt; text-decoration: none;border-radius: 50%;text-align:center;line-height:200px;}\
      </style>\
    </head>\
    <body>\
      <a href='/led/on'>LED ON</a>\
      <a href='/led/off'>LED OFF</a>\
      <form method='get' action='/led/setLightBrightness'>\
        <label>Led Brightness</label>\
        <input type='number' name='lightBrightness'><input type='submit'>\
      </form>\
    </body>\
    </html>";


String LEDOpenPage = "\
  <!DOCTYPE html>\
   <html>\
    <head>\
      <style>\
        body{ display: flex; flex-direction: column;}\
         a { background: #CEF2F4; width: 200px; height: 200px; margin: 20px; font-size: 30pt; text-decoration: none;border-radius: 50%;text-align:center;line-height:200px;}\
      </style>\
    </head>\
    <body>\
      <h1>LED OPEN</h1>\
      <a href='/'>Back</a>\
    </body>\
  </html>";

String LEDCLOSEPage = "\
  <!DOCTYPE html>\
   <html>\
    <head>\
      <style>\
        body{ display: flex; flex-direction: column;}\
        a { background: #CEF2F4; width: 200px; height: 200px; margin: 20px; font-size: 30pt; text-decoration: none;border-radius: 50%;text-align:center;line-height:200px;}\
      </style>\
    </head>\
    <body>\
      <h1>LED CLOSE</h1>\
      <a href='/'>Back</a>\
    </body>\
  </html>";
      

//------設定路徑------
void createWebServer()
{
  //初始頁面
  server.on("/", []() {
      server.send(200, "text/html", indexContent);
     // Serial.println("LED Off");
  });
  server.on("/led/on", []() {
    Serial.println("LED On");
    server.send(200, "text/html", LEDOpenPage );
    //add your code here
    digitalWrite(13,HIGH);
    //delay(1000);
    //String response = "{\"LED On Success\"}";
    //server.send(200, "application/json", response);
  });
  server.on("/led/off", []() {
    Serial.println("LED Off");
     digitalWrite(13,LOW);
     //delay(1000);
    server.send(200, "text/html", LEDCLOSEPage );
    //add your code here
    
    //String response = "{\"LED Off Success\"}";
    //server.send(200, "application/json", response);
  });
  server.on("/led/setLightBrightness", []() {
    //以server.arg&引數名稱，取得form傳來的資料
    int lightBrightness = server.arg("lightBrightness").toInt();
    //判斷條件    
    if(lightBrightness>=0 && lightBrightness<=1024){
      Serial.print("Set Brightness: ");
      Serial.println(lightBrightness);

      //add your code here
      ledcWrite(0, lightBrightness);  
      String response = "Set Brightness: " + (String)lightBrightness;
      server.send(200, "application/json", response);
    } 
    else {
      String response = "{\"Error\":\"over range 0-1024\"}";
      server.send(404, "application/json", response);
    }
  });


  
}






  

