ต่อไปนี้จะเป็น **ขั้นตอนการทำโปรเจคควบคุม LED ผ่านเว็บ** โดยใช้ **ESP8266** ตั้งแต่ต้นจนจบ โดยจะเริ่มตั้งแต่การซื้ออุปกรณ์ จนถึงการเขียนโค้ดและทดสอบระบบ

---

### **ขั้นตอนที่ 1: ซื้ออุปกรณ์**
เพื่อเริ่มต้นโปรเจคนี้ คุณจะต้องเตรียมอุปกรณ์ดังนี้:

1. **บอร์ด ESP8266 (NodeMCU หรือ Wemos D1 mini)**
   - คุณสามารถซื้อได้จากร้านออนไลน์ เช่น Lazada หรือ Shopee
   - ควรเลือก **NodeMCU** หรือ **Wemos D1 mini** เนื่องจากใช้งานง่ายและราคาถูก

2. **LED (Light Emitting Diode)** 1 ดวง
   - LED ขนาด 5mm สีแดง หรือเขียว

3. **Resistor (ตัวต้านทาน)** 220Ω หรือ 330Ω
   - ใช้เพื่อจำกัดกระแสไฟฟ้าผ่าน LED ป้องกันไม่ให้ LED เสีย

4. **สาย Jumper**
   - ใช้เชื่อมต่อระหว่าง ESP8266 และ LED

5. **สาย USB**
   - ใช้เชื่อมต่อ ESP8266 กับคอมพิวเตอร์เพื่ออัพโหลดโค้ด

---

### **ขั้นตอนที่ 2: เชื่อมต่ออุปกรณ์**

**การเชื่อมต่อ LED กับ ESP8266:**

1. เชื่อมต่อ **ขา Anode (ขั้วบวก)** ของ LED เข้ากับ **GPIO2 (D4)** ของ ESP8266
2. เชื่อมต่อ **ขา Cathode (ขั้วลบ)** ของ LED เข้ากับ **GND** ของ ESP8266 ผ่าน **Resistor** (220Ω หรือ 330Ω)

### **การเชื่อมต่อ:**
- **LED Anode (ขั้วบวก)** → **GPIO2 (D4)** ของ ESP8266
- **LED Cathode (ขั้วลบ)** → **GND** ของ ESP8266 ผ่าน **Resistor**

---

### **ขั้นตอนที่ 3: ติดตั้ง Arduino IDE**

1. **ดาวน์โหลด Arduino IDE**:
   - ไปที่เว็บไซต์ [Arduino IDE](https://www.arduino.cc/en/software) และดาวน์โหลดโปรแกรมสำหรับระบบปฏิบัติการที่คุณใช้ (Windows, macOS หรือ Linux)

2. **ติดตั้ง ESP8266 Board**:
   - เปิด Arduino IDE แล้วไปที่ **File** → **Preferences**.
   - ในช่อง **Additional Boards Manager URLs** ให้เพิ่ม URL ของ ESP8266:
     ```
     http://arduino.esp8266.com/stable/package_esp8266com_index.json
     ```
   - ไปที่ **Tools** → **Board** → **Boards Manager...**
   - ค้นหาคำว่า **ESP8266** แล้วติดตั้ง
   - เลือก **Board** เป็น **NodeMCU 1.0 (ESP-12E Module)** หรือ **Wemos D1 mini** ใน **Tools** → **Board**

---

### **ขั้นตอนที่ 4: เขียนโค้ด**

1. เปิด **Arduino IDE** ขึ้นมา
2. คัดลอกโค้ดตัวอย่างนี้และวางลงใน Arduino IDE:

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

// ตั้งค่า Wi-Fi
const char* ssid = "ชื่อ Wi-Fi ของคุณ";  // ชื่อ Wi-Fi
const char* password = "รหัสผ่าน Wi-Fi ของคุณ";  // รหัสผ่าน Wi-Fi

// กำหนดพอร์ตของ Web Server
ESP8266WebServer server(80);

// กำหนด LED pin
const int ledPin = 2;  // GPIO2 (D4) บน ESP8266

void setup() {
  // เริ่มต้น Serial communication
  Serial.begin(115200);

  // กำหนด LED Pin เป็น OUTPUT
  pinMode(ledPin, OUTPUT);

  // เชื่อมต่อ Wi-Fi
  WiFi.begin(ssid, password);

  // รอจนกว่าเชื่อมต่อ Wi-Fi เสร็จ
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("กำลังเชื่อมต่อ Wi-Fi...");
  }

  // เมื่อเชื่อมต่อเสร็จ
  Serial.println("เชื่อมต่อ Wi-Fi สำเร็จ!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());  // แสดง IP Address ของ ESP8266

  // กำหนด Route ของ Web Server
  server.on("/", HTTP_GET, []() {
    String html = "<html><body><h1>LED Control</h1>";
    html += "<p><a href=\"/on\"><button>เปิด LED</button></a></p>";
    html += "<p><a href=\"/off\"><button>ปิด LED</button></a></p>";
    html += "</body></html>";
    server.send(200, "text/html", html);  // ส่ง HTML ไปยังเว็บเบราว์เซอร์
  });

  // Route สำหรับเปิด LED
  server.on("/on", HTTP_GET, []() {
    digitalWrite(ledPin, HIGH);  // เปิด LED
    server.send(200, "text/html", "<h1>LED เปิดแล้ว</h1><a href=\"/\">กลับไปที่หน้าแรก</a>");
  });

  // Route สำหรับปิด LED
  server.on("/off", HTTP_GET, []() {
    digitalWrite(ledPin, LOW);  // ปิด LED
    server.send(200, "text/html", "<h1>LED ปิดแล้ว</h1><a href=\"/\">กลับไปที่หน้าแรก</a>");
  });

  // เริ่ม Web Server
  server.begin();
}

void loop() {
  server.handleClient();  // รับคำขอจากลูกค้าและตอบกลับ
}
```

#### อธิบายโค้ด:
- **Wi-Fi**: ฟังก์ชัน `WiFi.begin(ssid, password)` ใช้เชื่อมต่อกับเครือข่าย Wi-Fi
- **Web Server**: ใช้ ESP8266WebServer เพื่อรับคำขอ HTTP จากเว็บเบราว์เซอร์
- **LED**: ใช้ `digitalWrite(ledPin, HIGH)` เปิด LED และ `digitalWrite(ledPin, LOW)` ปิด LED
- **HTML**: สร้างหน้าเว็บที่มีปุ่ม "เปิด LED" และ "ปิด LED" ที่สามารถคลิกเพื่อควบคุม LED

---

### **ขั้นตอนที่ 5: อัพโหลดโค้ดไปยัง ESP8266**

1. เชื่อมต่อ **ESP8266** กับคอมพิวเตอร์ผ่าน **สาย USB**
2. เปิด **Arduino IDE** และเลือก **Board** เป็น **NodeMCU 1.0 (ESP-12E Module)** หรือ **Wemos D1 mini** ใน **Tools** → **Board**
3. เลือก **Port** ที่เชื่อมต่อกับ ESP8266 ใน **Tools** → **Port**
4. กด **Upload** เพื่ออัพโหลดโค้ดไปยัง ESP8266
5. รอจนกระทั่งโค้ดอัพโหลดเสร็จ

---

### **ขั้นตอนที่ 6: ทดสอบระบบ**

1. เปิด **Serial Monitor** ใน Arduino IDE และตรวจสอบว่า ESP8266 เชื่อมต่อกับ Wi-Fi หรือไม่
2. ใน **Serial Monitor** คุณจะเห็น **IP Address** ของ ESP8266
3. เปิด **เว็บเบราว์เซอร์** และพิมพ์ **IP Address** ของ ESP8266 ที่แสดงใน **Serial Monitor**
4. คุณจะเห็นหน้าเว็บที่มีปุ่ม "เปิด LED" และ "ปิด LED"
5. คลิกที่ปุ่ม "เปิด LED" เพื่อเปิด LED และคลิกที่ปุ่ม "ปิด LED" เพื่อปิด LED

---

### **ขั้นตอนที่ 7: พัฒนาฟีเจอร์เพิ่มเติม (ถ้าต้องการ)**

หากต้องการเพิ่มฟีเจอร์อื่น ๆ เช่น การควบคุม LED หลายดวงหรือการปรับแต่งหน้าเว็บ สามารถทำได้ตามขั้นตอนนี้:

- **เพิ่มการควบคุมหลาย LED**: สามารถเพิ่มฟังก์ชันควบคุม LED ตัวอื่น ๆ เช่น เพิ่ม **GPIO5 (D1)** หรือ **GPIO4 (D2)** และเขียนโค้ดเพิ่มเติม
- **ปรับปรุงหน้าตาเว็บ**: ใช้ **CSS** เพื่อทำให้หน้าเว็บสวยงามขึ้น หรือเพิ่มการแสดงสถานะของ LED
- **ใช้ MQTT**: ถ้าต้องการควบคุมจากอุปกรณ์อื่น ๆ ผ่าน **MQTT** โปรโตคอลที่ใช้ใน IoT

---

### **สรุป**
ตอนนี้คุณได้สร้างระบบ **ควบคุม LED ผ่านเว็บ** ด้วย **ESP8266** แล้ว โดยเริ่มตั้งแต่การซื้ออุปกรณ์, การเชื่อมต่อวงจร, การเขียนโค้ด, การอัพโหลดโค้ด, และการทดสอบระบบ คุณสามารถพัฒนาต่อและเพิ่มฟีเจอร์ใหม่ ๆ ได้ตามต้องการ
