int digit[4] = {28,27,22,14};   // PORTB values for digit selection (pins 9-12)
int number[10] = {126,12,182,158,204,218,250,14,254,222}; // segments for digits 0-9 (PORTD)

int d;
int n = 0;
int t0;
int t = 0;

int dig1, dig2, dig3, dig4;

bool start = false;      // true = counting, false = stopped
bool resetFlag = false;  // used for reset to 5555

void setup() {
  DDRD = 254;    // PD1-PD7 as output (segments), PD0 unused
  DDRB = 31;     // PB0-PB4 output (digit select), PB5 input (button)
  PORTB = 30;    // all digits off initially
  t = 5555;      // initial display "5555"
}

void loop() {
  if (digitalRead(13) == 1) {
    delay(50);   // debounce
    if (digitalRead(13) == 1) {
      if (!start) {
        // if stopped and currently showing 5555 -> start counting from 0
        if (t == 5555) {
          t = 0;
          start = true;
          t0 = millis();
        }
        // if stopped and not 5555 -> reset to 5555 (without starting)
        else {
          t = 5555;
          start = false;
        }
      }
      else {
        // if counting -> stop
        start = false;
      }
      delay(200); // prevent multiple triggers
    }
  }

  if (start == true) {
    unsigned long currentMillis = millis();
    unsigned long elapsed = currentMillis - t0;
    long hundredths = elapsed / 10;   // 1 = 0.01 sec
    if (hundredths > 5000) {          // max 50.00 sec
      hundredths = 5000;
      start = false;
    }
    t = (int)hundredths;
    out(t);
  } else {
    out(t);
  }
}

void out(int value) {
  int digits[4];
  if (value == 5555) {
    digits[0] = 5;
    digits[1] = 5;
    digits[2] = 5;
    digits[3] = 5;
  } else {
    int secs = value / 100;
    int cs = value % 100;
    digits[0] = secs / 10;        // tens of seconds
    digits[1] = secs % 10;        // seconds
    digits[2] = cs / 10;          // tenths
    digits[3] = cs % 10;          // hundredths
  }

  for (int d = 0; d < 4; d++) {
    PORTB = digit[d];
    int segData = number[digits[d]];

    // Мигание точки при счёте (каждые 300 мс инвертируем)
    static unsigned long lastBlink = 0;
    static bool pointState = false;
    if (start == true && millis() - lastBlink > 300) {
      pointState = !pointState;
      lastBlink = millis();
    }
    if (start == true && pointState == true) {
      // Assuming decimal point is bit 7 in PORTD (adjust if needed)
      segData = segData | 0b10000000;
    }

    PORTD = segData;
    delay(5);
  }
}
