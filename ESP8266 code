#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_Fingerprint.h>
#include <SoftwareSerial.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecure.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define BUZZER_PIN D3

const char* ssid      = "pwm";
const char* password  = "12345678";
const char* scriptURL = "SCRIPT_URL";

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
SoftwareSerial mySerial(D5, D6);
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);

bool enrollMode = false;
int  enrollID   = -1;

// ── GHOST FIX: confirm finger is really present ──────────────────
// Reads image 3 times — must get OK all 3 to be a real finger
bool isRealFinger() {
  int okCount = 0;
  for (int i = 0; i < 3; i++) {
    uint8_t p = finger.getImage();
    if (p == FINGERPRINT_OK) okCount++;
    else if (p == FINGERPRINT_NOFINGER) return false;
    delay(20);
  }
  return (okCount >= 3);
}

// ── Name map ─────────────────────────────────────────────────────
String getNameByID(int id) {
  switch (id) {
    case 31:  return "Mani";
    case 2:  return "Bob";
    case 3:  return "Charlie";
    case 4:  return "Mani";
    case 5:  return "Eve";
    case 6:  return "Frank";
    case 7:  return "Grace";
    case 8:  return "Henry";
    case 9:  return "Isla";
    case 34: return "DAVID";
    case 45: return "Ram";
    default: return "User_" + String(id);
  }
}

// ── Buzzer ───────────────────────────────────────────────────────
void beepSuccess() {
  tone(BUZZER_PIN, 1000, 150); delay(180);
  tone(BUZZER_PIN, 1400, 150); delay(180);
  tone(BUZZER_PIN, 1800, 200); delay(220);
  noTone(BUZZER_PIN);
}
void beepDenied() {
  tone(BUZZER_PIN, 400, 300); delay(350);
  tone(BUZZER_PIN, 300, 400); delay(450);
  noTone(BUZZER_PIN);
}
void beepStartup() {
  for (int f = 500; f <= 1500; f += 100) {
    tone(BUZZER_PIN, f, 60); delay(70);
  }
  noTone(BUZZER_PIN);
}
void beepScan() {
  tone(BUZZER_PIN, 800, 80); delay(100); noTone(BUZZER_PIN);
}
void beepEnroll() {
  tone(BUZZER_PIN, 1200, 100); delay(130);
  tone(BUZZER_PIN, 1200, 100); delay(130);
  noTone(BUZZER_PIN);
}

// ── OLED ─────────────────────────────────────────────────────────
void showMessage(String line1, String line2 = "", String line3 = "") {
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 8);  display.println(line1);
  display.setCursor(0, 28); display.println(line2);
  display.setCursor(0, 48); display.println(line3);
  display.display();
}

void showWaiting() {
  display.clearDisplay();
  int bx = 50, by = 8;
  display.drawRoundRect(bx, by, 28, 36, 5, SSD1306_WHITE);
  display.drawLine(bx+5, by+10, bx+22, by+10, SSD1306_WHITE);
  display.drawLine(bx+4, by+18, bx+23, by+18, SSD1306_WHITE);
  display.drawLine(bx+5, by+26, bx+22, by+26, SSD1306_WHITE);
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(10, 50);
  display.print("Place  Finger...");
  display.display();
}

void showEnrollWaiting(int id, int step) {
  display.clearDisplay();
  display.drawRoundRect(0, 0, 128, 64, 4, SSD1306_WHITE);
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(10, 6);  display.print("-- ENROLL MODE --");
  display.setCursor(6, 20);  display.print("ID: "); display.print(id);
  display.print("  Step: "); display.print(step); display.print("/2");
  display.setCursor(6, 34);
  if (step == 1) display.print("Place finger...");
  else           display.print("Place SAME finger");
  display.setCursor(6, 50);  display.print("R#"); display.print(id); display.print(" to cancel");
  display.display();
}

void showScanAnimation() {
  int cx = 64, cy = 32;
  for (int r = 5; r <= 28; r += 3) {
    display.clearDisplay();
    display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
    display.setCursor(20, 56); display.print("Scanning...");
    display.drawCircle(cx, cy, r, SSD1306_WHITE);
    if (r > 8)  display.drawCircle(cx, cy, r-6,  SSD1306_WHITE);
    if (r > 14) display.drawCircle(cx, cy, r-12, SSD1306_WHITE);
    display.fillCircle(cx, cy, 3, SSD1306_WHITE);
    display.display(); delay(60);
  }
}

void showGrantedAnimation(int id, String name) {
  int cx = 64, cy = 32;
  for (int r = 2; r <= 72; r += 5) {
    display.clearDisplay();
    display.fillCircle(cx, cy, r, SSD1306_WHITE);
    display.display(); delay(18);
  }
  display.clearDisplay();
  display.fillRect(0, 0, 128, 64, SSD1306_WHITE);
  for (int t = 0; t <= 2; t++) {
    display.drawLine(38+t, 30, 50+t, 44, SSD1306_BLACK);
    display.drawLine(50+t, 44, 74+t, 18, SSD1306_BLACK);
  }
  display.display(); delay(400);

  display.clearDisplay();
  display.drawRoundRect(2, 2, 124, 60, 6, SSD1306_WHITE);
  display.drawRoundRect(4, 4, 120, 56, 4, SSD1306_WHITE);
  for (int t = 0; t <= 1; t++) {
    display.drawLine(22+t, 22, 34+t, 36, SSD1306_WHITE);
    display.drawLine(34+t, 36, 54+t, 10, SSD1306_WHITE);
  }
  display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
  display.setCursor(60, 10); display.print("MARKED!");
  display.setCursor(60, 26); display.print("ID: "); display.print(id);
  display.setCursor(60, 42);
  if (name.length() > 8) name = name.substring(0, 8);
  display.print(name);
  display.display();
}

void showDeniedAnimation() {
  for (int b = 0; b < 3; b++) {
    display.clearDisplay();
    display.fillRect(0, 0, 128, 64, SSD1306_WHITE);
    display.display(); delay(80);
    display.clearDisplay(); display.display(); delay(80);
  }
  display.clearDisplay();
  display.drawRoundRect(2, 2, 124, 60, 6, SSD1306_WHITE);
  for (int t = -1; t <= 1; t++) {
    display.drawLine(30+t, 12, 98+t, 46, SSD1306_WHITE);
    display.drawLine(98+t, 12, 30+t, 46, SSD1306_WHITE);
  }
  display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
  display.setCursor(22, 51); display.print("ACCESS  DENIED");
  display.display();
}

void showSendingAnimation() {
  for (int dot = 1; dot <= 3; dot++) {
    display.clearDisplay();
    display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
    display.setCursor(20, 20); display.print("Sending data");
    for (int d = 0; d < dot; d++) display.print(".");
    for (int b = 0; b < 4; b++) {
      int bh = (b+1)*6, bx = 44+b*12, by2 = 50-bh;
      if (b < dot) display.fillRect(bx, by2, 8, bh, SSD1306_WHITE);
      else         display.drawRect(bx, by2, 8, bh, SSD1306_WHITE);
    }
    display.display(); delay(400);
  }
}

// ── List stored IDs ──────────────────────────────────────────────
void listStoredIDs() {
  Serial.println("=================================");
  Serial.println("       STORED FINGERPRINT IDs    ");
  Serial.println("=================================");
  finger.getTemplateCount();
  Serial.print("Total: "); Serial.println(finger.templateCount);
  Serial.println("---------------------------------");
  int found = 0;
  for (int i = 1; i <= 127; i++) {
    if (finger.loadModel(i) == FINGERPRINT_OK) {
      Serial.print("  ID #");
      if (i < 10) Serial.print("0");
      Serial.print(i); Serial.print("  →  ");
      Serial.println(getNameByID(i));
      found++;
    }
  }
  if (found == 0) Serial.println("  No fingerprints stored.");
  Serial.println("=================================");

  display.clearDisplay();
  display.drawRoundRect(0, 0, 128, 64, 4, SSD1306_WHITE);
  display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
  display.setCursor(10, 6);  display.print("Stored IDs: "); display.print(found);
  display.setCursor(6, 22);  display.print("Check Serial");
  display.setCursor(6, 36);  display.print("Monitor for");
  display.setCursor(6, 50);  display.print("full list...");
  display.display(); delay(3000);
}

// ── Remove fingerprint ───────────────────────────────────────────
void removeFingerprint(int id) {
  Serial.print("Removing ID #"); Serial.println(id);
  showMessage("Removing...", "ID: " + String(id), "Please wait");
  uint8_t p = finger.deleteModel(id);
  if (p == FINGERPRINT_OK) {
    Serial.println("Deleted!");
    display.clearDisplay();
    display.drawRoundRect(2, 2, 124, 60, 6, SSD1306_WHITE);
    display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
    display.setCursor(22, 14); display.print("ID REMOVED!");
    display.drawLine(4, 26, 122, 26, SSD1306_WHITE);
    display.setCursor(10, 34); display.print("ID #"); display.print(id); display.print(" deleted");
    display.setCursor(10, 48); display.print(getNameByID(id));
    display.display();
    beepDenied(); delay(2500);
  } else {
    Serial.println("Delete failed!");
    showMessage("Delete Failed!", "ID #" + String(id), "Not found?");
    beepDenied(); delay(2000);
  }
}

// ── Enroll fingerprint ───────────────────────────────────────────
bool enrollFingerprint(int id) {
  Serial.println("---------------------------------");
  Serial.print("Enrolling ID #"); Serial.println(id);

  // SCAN 1
  showEnrollWaiting(id, 1);
  Serial.println("Place finger (scan 1)...");

  uint8_t p = -1;
  unsigned long t = millis();
  while (p != FINGERPRINT_OK) {
    if (Serial.available()) {
      String cmd = Serial.readStringUntil('\n');
      cmd.trim();
      if (cmd.startsWith("R#") || cmd == "A") {
        Serial.println("Enroll cancelled.");
        showMessage("Enroll", "Cancelled");
        enrollMode = false; enrollID = -1;
        delay(1500); return false;
      }
    }
    p = finger.getImage();
    if (p == FINGERPRINT_NOFINGER) {
      if (millis() - t > 30000) {
        showMessage("Timeout!", "Enroll cancelled");
        enrollMode = false; enrollID = -1;
        delay(2000); return false;
      }
      continue;
    }
    if (p != FINGERPRINT_OK) continue;
  }

  beepEnroll();
  showMessage("Finger Detected", "Processing...");
  p = finger.image2Tz(1);
  if (p != FINGERPRINT_OK) {
    showMessage("Convert Failed", "Try again");
    beepDenied(); delay(2000); return false;
  }

  showMessage("Scan 1 OK!", "Remove finger", "");
  beepSuccess(); delay(1500);

  p = 0;
  while (p != FINGERPRINT_NOFINGER) p = finger.getImage();

  // SCAN 2
  showEnrollWaiting(id, 2);
  Serial.println("Place SAME finger (scan 2)...");

  p = -1; t = millis();
  while (p != FINGERPRINT_OK) {
    if (Serial.available()) {
      String cmd = Serial.readStringUntil('\n');
      cmd.trim();
      if (cmd.startsWith("R#") || cmd == "A") {
        Serial.println("Enroll cancelled.");
        showMessage("Enroll", "Cancelled");
        enrollMode = false; enrollID = -1;
        delay(1500); return false;
      }
    }
    p = finger.getImage();
    if (p == FINGERPRINT_NOFINGER) {
      if (millis() - t > 30000) {
        showMessage("Timeout!", "Try again");
        enrollMode = false; enrollID = -1;
        delay(2000); return false;
      }
      continue;
    }
    if (p != FINGERPRINT_OK) continue;
  }

  beepEnroll();
  showMessage("Finger Detected", "Processing...");
  p = finger.image2Tz(2);
  if (p != FINGERPRINT_OK) {
    showMessage("Convert Failed", "Try again");
    beepDenied(); delay(2000); return false;
  }

  showMessage("Creating", "Model...");
  p = finger.createModel();
  if (p == FINGERPRINT_ENROLLMISMATCH) {
    showMessage("MISMATCH!", "Use SAME finger", "Try again");
    beepDenied(); delay(2500); return false;
  } else if (p != FINGERPRINT_OK) {
    showMessage("Model Failed", "Try again");
    beepDenied(); delay(2000); return false;
  }

  showMessage("Storing...", "ID: " + String(id));
  p = finger.storeModel(id);
  if (p == FINGERPRINT_OK) {
    finger.getTemplateCount();
    display.clearDisplay();
    display.drawRoundRect(2, 2, 124, 60, 6, SSD1306_WHITE);
    display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
    display.setCursor(18, 6);  display.print("ENROLLED!");
    display.drawLine(4, 18, 122, 18, SSD1306_WHITE);
    display.setCursor(6, 26);  display.print("ID   : #"); display.print(id);
    display.setCursor(6, 38);  display.print("Name : "); display.print(getNameByID(id));
    display.setCursor(6, 52);  display.print("Total: "); display.print(finger.templateCount);
    display.display();
    beepSuccess(); delay(3000);
    enrollMode = false; enrollID = -1;
    return true;
  } else {
    showMessage("Store Failed!", "Try again");
    beepDenied(); delay(2000); return false;
  }
}

// ── Handle Serial commands ───────────────────────────────────────
void handleSerial() {
  if (!Serial.available()) return;
  String cmd = Serial.readStringUntil('\n');
  cmd.trim(); cmd.toUpperCase();
  Serial.println("CMD: " + cmd);

  if (cmd == "L") { listStoredIDs(); return; }

  if (cmd == "A") {
    enrollMode = true; enrollID = -1;
    Serial.println("ENROLL MODE — Send #<id>");
    showMessage("ENROLL MODE", "Send #<id>", "e.g. #5");
    return;
  }

  if (cmd.startsWith("R#")) {
    int id = cmd.substring(2).toInt();
    if (id >= 1 && id <= 127) removeFingerprint(id);
    else Serial.println("Invalid ID");
    enrollMode = false; enrollID = -1;
    return;
  }

  if (cmd.startsWith("#")) {
    int id = cmd.substring(1).toInt();
    if (id >= 1 && id <= 127) {
      enrollID = id; enrollMode = true;
      enrollFingerprint(id);
    } else Serial.println("Invalid ID");
    return;
  }

  Serial.println("Commands: A | #<id> | R#<id> | L");
}

// ── Send attendance ──────────────────────────────────────────────
void sendAttendance(int id, String name) {
  showSendingAnimation();
  if (WiFi.status() != WL_CONNECTED) {
    showMessage("WiFi Lost!", "Reconnecting...");
    WiFi.begin(ssid, password);
    int tries = 0;
    while (WiFi.status() != WL_CONNECTED && tries < 20) {
      delay(500); tries++;
    }
    if (WiFi.status() != WL_CONNECTED) {
      showMessage("WiFi Failed!", "Not logged!", "Check hotspot");
      beepDenied(); delay(2000); return;
    }
  }

  WiFiClientSecure client;
  client.setInsecure();
  HTTPClient http;
  http.begin(client, scriptURL);
  http.addHeader("Content-Type", "application/json");
  http.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS);

  String payload = "{\"id\":" + String(id) +
                   ",\"name\":\"" + name +
                   "\",\"status\":\"Present\"}";

  Serial.println("Payload: " + payload);
  int httpCode = http.POST(payload);
  Serial.print("HTTP: "); Serial.println(httpCode);

  if (httpCode == 200 || httpCode == 302) {
    display.clearDisplay();
    display.drawRoundRect(2, 2, 124, 60, 6, SSD1306_WHITE);
    display.setTextSize(1); display.setTextColor(SSD1306_WHITE);
    display.setCursor(18, 8);  display.print("ATTENDANCE SAVED");
    display.drawLine(4, 20, 122, 20, SSD1306_WHITE);
    display.setCursor(10, 26); display.print("Name: "); display.print(name);
    display.setCursor(10, 38); display.print("ID  : "); display.print(id);
    display.setCursor(10, 50); display.print("Status: Present");
    display.display();
    beepSuccess();
  } else {
    showMessage("Log Failed!", "HTTP:" + String(httpCode), "Try again");
    beepDenied();
  }
  http.end(); delay(3000);
}

// ── Fingerprint scan (GHOST BUG FIXED) ───────────────────────────
int getFingerprintID() {

  // ── FIX 1: confirm real finger with triple check ─────────────
  if (!isRealFinger()) return -1;

  // ── FIX 2: small delay to let sensor stabilise ───────────────
  delay(50);

  // ── FIX 3: get a fresh clean image after confirmation ────────
  uint8_t p = finger.getImage();
  if (p != FINGERPRINT_OK) return -1;

  beepScan();
  showScanAnimation();

  p = finger.image2Tz();
  if (p != FINGERPRINT_OK) {
    showMessage("Convert Failed", "Try again");
    return -1;
  }

  p = finger.fingerFastSearch();
  if (p == FINGERPRINT_OK) {
    // ── FIX 4: ignore low confidence matches (noise/ghost) ─────
    if (finger.confidence < 50) {
      Serial.println("Low confidence ghost — ignored");
      Serial.print("Confidence was: "); Serial.println(finger.confidence);
      return -1;
    }
    Serial.print("Match ID: ");    Serial.println(finger.fingerID);
    Serial.print("Confidence: "); Serial.println(finger.confidence);
    return finger.fingerID;
  }

  // ── FIX 5: wait for finger to fully lift before denied anim ──
  delay(100);
  p = finger.getImage();
  if (p == FINGERPRINT_NOFINGER) {
    // it was already gone — was likely just noise, skip denied
    Serial.println("Ghost detected and ignored");
    return -1;
  }

  Serial.println("No Match");
  beepDenied();
  showDeniedAnimation();

  // ── FIX 6: wait for finger lift after denied ─────────────────
  while (finger.getImage() != FINGERPRINT_NOFINGER) delay(50);
  delay(500);

  return -1;
}

// ── Setup ────────────────────────────────────────────────────────
void setup() {
  Serial.begin(115200);
  pinMode(BUZZER_PIN, OUTPUT);
  delay(500);

  Wire.begin(D2, D1);
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED Failed"); while(1);
  }

  for (int x = 0; x <= 128; x += 4) {
    display.clearDisplay();
    display.fillRect(0, 0, x, 64, SSD1306_WHITE);
    display.display(); delay(10);
  }
  display.clearDisplay();
  display.setTextSize(2); display.setTextColor(SSD1306_WHITE);
  display.setCursor(4, 10);  display.print("ATTEND");
  display.setCursor(16, 38); display.print("SYSTEM");
  display.display();
  beepStartup(); delay(1500);

  showMessage("Connecting WiFi", ssid, "Please wait...");
  WiFi.begin(ssid, password);
  int tries = 0;
  while (WiFi.status() != WL_CONNECTED && tries < 30) {
    delay(500); Serial.print("."); tries++;
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWiFi Connected: " + WiFi.localIP().toString());
    showMessage("WiFi Connected!", WiFi.localIP().toString(), "");
    tone(BUZZER_PIN, 1200, 300); delay(1500);
  } else {
    showMessage("WiFi FAILED!", "Check hotspot:", ssid);
    beepDenied(); delay(2000);
  }

  mySerial.begin(57600);
  finger.begin(57600);

  if (finger.verifyPassword()) {
    Serial.println("Fingerprint Sensor Ready");
    showMessage("Sensor Ready!", "", "");
  } else {
    Serial.println("Sensor Not Found");
    showMessage("Sensor Error!", "Check wiring");
    while(1);
  }

  finger.getTemplateCount();
  Serial.println("=================================");
  Serial.println("   ATTENDANCE SYSTEM READY");
  Serial.println("=================================");
  Serial.println("Commands: A | #<id> | R#<id> | L");
  Serial.println("=================================");
  showMessage("Stored Prints:", String(finger.templateCount), "System Ready!");
  delay(2000);
}

// ── Main loop ────────────────────────────────────────────────────
void loop() {
  handleSerial();

  if (enrollMode && enrollID == -1) {
    showMessage("ENROLL MODE", "Send #<id>", "via Serial");
    delay(300);
    return;
  }

  if (!enrollMode) {
    showWaiting();
    int id = getFingerprintID();
    if (id > 0) {
      String name = getNameByID(id);
      Serial.println("----------------");
      Serial.print("Matched ID: "); Serial.println(id);
      Serial.print("Name: ");       Serial.println(name);
      showGrantedAnimation(id, name);
      delay(500);
      sendAttendance(id, name);
    }
  }
}
