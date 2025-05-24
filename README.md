#include <LiquidCrystal.h>
#include <Keyboard.h>

// === LCD (LiquidCrystal) ===
LiquidCrystal lcd(2, 3, 4, 5, 6, 7); // RS, E, D4, D5, D6, D7

// === Przyciski i buzzer ===
const int buttonUpPin = 9;
const int buttonDownPin = 10;
const int buttonOkPin = 11;
const int buttonBackPin = 12; // opcjonalnie "wstecz"
const int buzzerPin = 8;

bool mute = false;
bool menuMode = true;
int menuIndex = 0;
int menuOffset = 0;
unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 200;

// === Własne znaki na LCD ===
byte fullBlock[8] = {B11111,B11111,B11111,B11111,B11111,B11111,B11111,B11111};

// === Menu główne ===
const char* menuItems[] = {
  "WiFi Scan", "Info", "Timer", "Password Gen", "Mute",
  "Brute Force", "ROT13", "BLE Spam", "Keylogger", "WiFi Jammer",
  "Phishing Sim", "Pass Manager", "Fake Hack", "ProgressBar",
  "Counter", "Matrix", "RAM Info"
};
const int menuLength = sizeof(menuItems) / sizeof(menuItems[0]);

// === BruteForce ===
// Rozszerzona wordlista (ponad 200 haseł)
const char* wordlist[] = {
  "123456", "password", "123456789", "12345", "12345678", "qwerty", "1234567", "111111", "123123", "abc123",
  "1234567890", "1234", "iloveyou", "123", "1q2w3e4r", "000000", "qwertyuiop", "123321", "password1", "monkey",
  "dragon", "letmein", "football", "baseball", "master", "hello", "freedom", "whatever", "qazwsx", "trustno1",
  "654321", "jordan", "superman", "michael", "shadow", "sunshine", "123qwe", "hottie", "loveme", "zaq12wsx",
  "password123", "batman", "696969", "qwerty123", "123654", "superuser", "passw0rd", "starwars", "654321", "7777777",
  "ashley", "bailey", "access", "flower", "mustang", "maggie", "buster", "soccer", "harley", "hunter",
  "jessica", "ginger", "michelle", "pepper", "daniel", "summer", "ashley", "nicole", "chelsea", "biteme",
  "matthew", "yankees", "martin", "tigger", "charlie", "robert", "thomas", "hannah", "amanda", "lovely",
  "joshua", "maggie", "cheese", "computer", "bubbles", "samantha", "patrick", "justin", "orange", "banana",
  "donald", "princess", "qwert", "killer", "hockey", "george", "cookie", "naruto", "pokemon", "qwertyui",
  // Dodatkowe hasła
  "welcome123", "admin123", "welcome1", "letmein1", "password12", "football1", "baseball1", "soccer1", "welcome2", "admin1",
  "admin1234", "adminadmin", "adminpassword", "admin12345", "adminpass", "admin123456", "admin1234567", "admin12345678", "admin123456789", "admin1234567890",
  "test123", "test1234", "test12345", "test123456", "test1234567", "test12345678", "test123456789", "test1234567890", "testpassword", "testpass",
  "login123", "login1234", "login12345", "login123456", "login1234567", "login12345678", "login123456789", "login1234567890", "loginpassword", "loginpass",
  "user123", "user1234", "user12345", "user123456", "user1234567", "user12345678", "user123456789", "user1234567890", "userpassword", "userpass",
  "guest123", "guest1234", "guest12345", "guest123456", "guest1234567", "guest12345678", "guest123456789", "guest1234567890", "guestpassword", "guestpass",
  "root123", "root1234", "root12345", "root123456", "root1234567", "root12345678", "root123456789", "root1234567890", "rootpassword", "rootpass",
  "demo123", "demo1234", "demo12345", "demo123456", "demo1234567", "demo12345678", "demo123456789", "demo1234567890", "demopassword", "demopass",
  "temp123", "temp1234", "temp12345", "temp123456", "temp1234567", "temp12345678", "temp123456789", "temp1234567890", "temppassword", "temppass",
  "work123", "work1234", "work12345", "work123456", "work1234567", "work12345678", "work123456789", "work1234567890", "workpassword", "workpass",
  "home123", "home1234", "home12345", "home123456", "home1234567", "home12345678", "home123456789", "home1234567890", "homepassword", "homepass",
  "school123", "school1234", "school12345", "school123456", "school1234567", "school12345678", "school123456789", "school1234567890", "schoolpassword", "schoolpass",
  "secret123", "secret1234", "secret12345", "secret123456", "secret1234567", "secret12345678", "secret123456789", "secret1234567890", "secretpassword", "secretpass",
  "secure123", "secure1234", "secure12345", "secure123456", "secure1234567", "secure12345678", "secure123456789", "secure1234567890", "securepassword", "securepass"
};
const int wordlistCount = sizeof(wordlist) / sizeof(wordlist[0]);
int bruteForceLength = 4;

// === Funkcje pomocnicze ===
bool isButtonPressed(int pin) { return digitalRead(pin) == LOW; }
void beep(int t = 100) { if (!mute) { digitalWrite(buzzerPin, HIGH); delay(t); digitalWrite(buzzerPin, LOW); } }
void clearLine(int row) { lcd.setCursor(0, row); lcd.print("                "); lcd.setCursor(0, row); }

// === Wyświetlanie menu z podświetleniem i przewijaniem ===
void showMenu(int selected, int offset) {
  lcd.clear();
  for (int i = 0; i < 2 && i + offset < menuLength; i++) {
    lcd.setCursor(0, i);
    if (i + offset == selected) lcd.write(byte(0)); // fullBlock
    else lcd.print(" ");
    lcd.print(menuItems[i + offset]);
  }
}

// === Funkcje HackBox PRO MAX ===

void wifiScan() {
  lcd.clear();
  lcd.print("Scanning WiFi...");
  delay(1000);
  lcd.clear();
  lcd.print("WiFi Scan");
  lcd.setCursor(0,1);
  lcd.print("Not supported");
  delay(2000);
}

void info() {
  lcd.clear();
  lcd.print("HackBox PRO MAX");
  lcd.setCursor(0, 1);
  lcd.print("by ORION");
  delay(2000);
}

void timer() {
  lcd.clear();
  lcd.print("Czas:");
  for (int i = 0; i < 10; i++) {
    lcd.setCursor(6, 0);
    lcd.print("  ");
    lcd.setCursor(6, 0);
    lcd.print(i);
    delay(1000);
  }
}

void passwordGen() {
  lcd.clear();
  lcd.print("Haslo: ");
  const char chars[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  for (int i = 0; i < 4; i++) {
    lcd.print(chars[random(0, sizeof(chars) - 1)]);
    delay(500);
  }
  delay(1000);
}

void toggleMute() {
  mute = !mute;
  lcd.clear();
  lcd.print(mute ? "Mute: ON" : "Mute: OFF");
  delay(1500);
}

String randomPassword(int length) {
  const char chars[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  String pwd = "";
  for (int i = 0; i < length; i++)
    pwd += chars[random(0, sizeof(chars) - 1)];
  return pwd;
}

void bruteRandom() {
  lcd.clear();
  lcd.print("Brute: Random");
  delay(1000);
  for (int i = 0; i < 10; i++) {
    String pwd = randomPassword(bruteForceLength);
    lcd.clear();
    lcd.setCursor(0, 0);
    int percent = (i * 100) / 10;
    for (int j = 0; j < map(percent, 0, 100, 0, 16); j++) lcd.write(byte(0));
    lcd.setCursor(0, 1);
    lcd.print(pwd);
    Keyboard.print(pwd);   // <-- wpisuje hasło na komputerze
    Keyboard.write('\n');  // <-- zatwierdza Enterem
    delay(1000);           // <-- dłuższy delay, byś zdążył zobaczyć
  }
  lcd.clear();
  lcd.print("Done!");
  delay(1000);
}

void bruteWordlist() {
  lcd.clear();
  lcd.print("Brute: Wordlist");
  delay(1000);
  for (int i = 0; i < wordlistCount; i++) {
    lcd.clear();
    lcd.setCursor(0, 0);
    int percent = (i * 100) / wordlistCount;
    for (int j = 0; j < map(percent, 0, 100, 0, 16); j++) lcd.write(byte(0));
    lcd.setCursor(0, 1);
    lcd.print(wordlist[i]);
    Keyboard.print(wordlist[i]);   // <-- wpisuje hasło na komputerze
    Keyboard.write('\n');          // <-- zatwierdza Enterem
    delay(1000);                   // <-- dłuższy delay, byś zdążył zobaczyć
  }
  lcd.clear();
  lcd.print("Done!");
  delay(1000);
}

void bruteForce() {
  lcd.clear();
  lcd.print("Brute Len:");
  lcd.setCursor(11, 0);
  lcd.print(bruteForceLength);
  lcd.setCursor(0, 1);
  lcd.print("UP=Rnd DN=List");

  while (true) {
    if (millis() - lastDebounceTime > debounceDelay) {
      if (isButtonPressed(buttonUpPin)) {
        beep();
        bruteRandom();
        break;
      } else if (isButtonPressed(buttonDownPin)) {
        beep();
        bruteWordlist();
        break;
      } else if (isButtonPressed(buttonOkPin)) {
        bruteForceLength++;
        if (bruteForceLength > 8)
          bruteForceLength = 3;
        lcd.setCursor(11, 0);
        lcd.print("  ");
        lcd.setCursor(11, 0);
        lcd.print(bruteForceLength);
        beep();
        lastDebounceTime = millis();
      }
    }
  }
}

void rot13() {
  lcd.clear();
  lcd.print("ROT13:");
  String msg = "sigma";
  delay(1500);
  String res = "";
  for (char c : msg) {
    if (c >= 'a' && c <= 'z')
      c = (c - 'a' + 13) % 26 + 'a';
    else if (c >= 'A' && c <= 'Z')
      c = (c - 'A' + 13) % 26 + 'A';
    res += c;
  }
  lcd.clear();
  lcd.print(res);
  delay(2000);
}

void bleSpam() {
  lcd.clear();
  lcd.print("BLE Spam");
  lcd.setCursor(0,1);
  lcd.print("Not supported");
  delay(2000);
}

void keyloggerMode() {
  lcd.clear();
  lcd.print("Keylogger");
  lcd.setCursor(0,1);
  lcd.print("Not supported");
  delay(2000);
}

void wifiJammer() {
  lcd.clear();
  lcd.print("WiFi Jammer");
  lcd.setCursor(0, 1);
  lcd.print("Not supported");
  delay(3000);
}

void phishingSim() {
  lcd.clear();
  lcd.print("Phishing Sim");
  lcd.setCursor(0,1);
  lcd.print("Not supported");
  delay(2000);
}

void passwordManager() {
  lcd.clear();
  lcd.print("Pass Manager");
  lcd.setCursor(0,1);
  lcd.print("No Data Yet");
  delay(2000);
}

void fakeHackAnimation() {
  lcd.clear();
  lcd.print("Hacking...");
  for (int i = 0; i < 10; i++) {
    lcd.setCursor(random(0, 16), random(0, 2));
    lcd.print((char)random(33, 127));
    delay(200);
  }
  lcd.clear();
  lcd.print("Access Granted!");
  delay(1500);
}

void progressBarDemo() {
  lcd.clear();
  lcd.print("ProgressBar:");
  for (int i = 0; i <= 16; i++) {
    lcd.setCursor(0, 1);
    for (int j = 0; j < i; j++) lcd.write(byte(0));
    for (int j = i; j < 16; j++) lcd.print(" ");
    delay(120);
  }
  lcd.setCursor(0, 1);
  lcd.print("COMPLETE!      ");
  delay(1200);
}

void counter() {
  int cnt = 0;
  lcd.clear();
  lcd.print("Counter:");
  lcd.setCursor(0,1);
  lcd.print(cnt);
  while (true) {
    if (isButtonPressed(buttonUpPin)) {
      cnt++;
      lcd.setCursor(0,1); lcd.print("    "); lcd.setCursor(0,1); lcd.print(cnt);
      beep();
      delay(200);
    }
    if (isButtonPressed(buttonDownPin)) {
      cnt--;
      lcd.setCursor(0,1); lcd.print("    "); lcd.setCursor(0,1); lcd.print(cnt);
      beep();
      delay(200);
    }
    if (isButtonPressed(buttonOkPin)) break;
  }
}

void matrixEffect() {
  lcd.clear();
  for (int t = 0; t < 30; t++) {
    for (int i = 0; i < 16; i++) {
      lcd.setCursor(i, random(0,2));
      lcd.print((char)random(33,127));
    }
    delay(100);
  }
  lcd.clear();
  lcd.print("MATRIX END");
  delay(1000);
}

void ramInfo() {
  lcd.clear();
  lcd.print("RAM Info:");
  lcd.setCursor(0,1);
  lcd.print("Not supported");
  delay(2000);
}

// === SETUP i LOOP ===
void setup() {
  pinMode(buttonUpPin, INPUT_PULLUP);
  pinMode(buttonDownPin, INPUT_PULLUP);
  pinMode(buttonOkPin, INPUT_PULLUP);
  if (buttonBackPin != -1) pinMode(buttonBackPin, INPUT_PULLUP);
  pinMode(buzzerPin, OUTPUT);

  Serial.begin(9600);
  Keyboard.begin();

  lcd.begin(16, 2);
  lcd.createChar(0, fullBlock);

  // Powitanie "HackBox PRO MAX by ORION"
  lcd.clear();
  lcd.print("HackBox PRO MAX");
  lcd.setCursor(0,1);
  lcd.print("by ORION");
  delay(2000);

  showMenu(menuIndex, menuOffset);
}

void loop() {
  if (millis() - lastDebounceTime > debounceDelay) {
    if (isButtonPressed(buttonUpPin)) {
      beep();
      menuIndex--;
      if (menuIndex < 0) menuIndex = menuLength - 1;
      menuOffset = (menuIndex / 2) * 2;
      showMenu(menuIndex, menuOffset);
      lastDebounceTime = millis();
    }
    if (isButtonPressed(buttonDownPin)) {
      beep();
      menuIndex++;
      if (menuIndex >= menuLength) menuIndex = 0;
      menuOffset = (menuIndex / 2) * 2;
      showMenu(menuIndex, menuOffset);
      lastDebounceTime = millis();
    }
    if (isButtonPressed(buttonOkPin)) {
      beep();
      lcd.clear();
      switch (menuIndex) {
        case 0: wifiScan(); break;
        case 1: info(); break;
        case 2: timer(); break;
        case 3: passwordGen(); break;
        case 4: toggleMute(); break;
        case 5: bruteForce(); break;
        case 6: rot13(); break;
        case 7: bleSpam(); break;
        case 8: keyloggerMode(); break;
        case 9: wifiJammer(); break;
        case 10: phishingSim(); break;
        case 11: passwordManager(); break;
        case 12: fakeHackAnimation(); break;
        case 13: progressBarDemo(); break;
        case 14: counter(); break;
        case 15: matrixEffect(); break;
        case 16: ramInfo(); break;
        default:
          lcd.print("Brak funkcji");
          delay(1000);
      }
      showMenu(menuIndex, menuOffset);
      lastDebounceTime = millis();
    }
  }
}
