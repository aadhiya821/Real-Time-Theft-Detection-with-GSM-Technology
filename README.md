#include <SoftwareSerial.h>
#include "Adafruit_FONA.h"
#include <string.h>

// GSM module pins
#define FONA_RX 2
#define FONA_TX 3
#define FONA_RST 4

// PIR Sensor pin
#define PIR_PIN 5

SoftwareSerial fonaSS(FONA_TX, FONA_RX);
Adafruit_FONA fona = Adafruit_FONA(FONA_RST);

// Enter your phone number with country code
char PHONE_1[21] = "+91XXXXXXXXXX";

// Alert message
char theftalertmessage[141] = "Theft Alert in Home! Motion Detected.";

// Sensor variable
int pirsensor = 0;

void setup()
{
  pinMode(PIR_PIN, INPUT);

  Serial.begin(115200);
  Serial.println("Initializing GSM Module...");

  delay(5000);

  fonaSS.begin(9600);

  if (!fona.begin(fonaSS))
  {
    Serial.println("Couldn't find GSM Module");
    while (1);
  }

  Serial.println("GSM Module Ready");

  // SMS configuration
  fona.print("AT+CSMP=17,167,0,0\r");
}

void loop()
{
  pirsensor = digitalRead(PIR_PIN);

  Serial.print("Sensor Value: ");
  Serial.println(pirsensor);

  if (pirsensor == HIGH)
  {
    Serial.println("⚠ Motion Detected - Theft Alert");

    make_call(PHONE_1);
    send_sms(PHONE_1);

    delay(30000); // Prevent repeated alerts
  }
  else
  {
    Serial.println("Safe");
  }

  delay(1000);
}

void send_sms(char number[])
{
  if (strlen(number) > 0)
  {
    Serial.print("Sending SMS to: ");
    Serial.println(number);

    if (fona.sendSMS(number, theftalertmessage))
      Serial.println("SMS Sent Successfully");
    else
      Serial.println("SMS Failed");
  }
}

void make_call(char number[])
{
  if (strlen(number) > 0)
  {
    Serial.print("Calling: ");
    Serial.println(number);

    fona.println(String("ATD") + number + ";");

    delay(20000);  // call for 20 seconds

    fona.println("ATH"); // hang up

    Serial.println("Call Ended");
  }
}
