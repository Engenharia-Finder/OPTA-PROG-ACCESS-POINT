# Overview
The programming aims to create an access point in OPTA, serving as a type of Wi-Fi "router" with username and password.
It is possible to control two output LEDs and read an OPTA analog port on a web page.
To access the website, you must connect your cell phone, tablet or computer to the Wi-Fi created by OPTA with the following credentials:
Network name: yourNetwork
Network password: yourPassword

After the device is connected to the network, you need to enter the IP 192.168.3.1 in the browser


# Goals
Learn how to create an access point (as if it were a router) on an OPTA so that a cell phone, computer or other OPTA can connect to this generated network.

# Finder OPTA and the Access Point

Creating an access point in OPTA is the same thing as creating a Wi-Fi network, but without internet access.
This type of application serves several functions, one of the main ones being the creation of a network without internet access for security reasons or even a bad signal.


# Network configuration
In the code below we include the WiFi.h library and then configure the network name and password:

```
#include <WiFi.h>

///////please enter your sensitive data in the Secret tab/arduino_secrets.h
char ssid[] = "yourNetwork";        // your network SSID (name)
char pass[] = "yourPassword";        // your network password (use for WPA, or use as key for WEP)
int keyIndex = 0;                 // your network key Index number (needed only for WEP)
```
# Creating a variable to read Wi-Fi Status

```
int status = WL_IDLE_STATUS;
...
```

# Remaining code

The code below is used to configure the output LED and also configure the HTML blocks that involve this status on the serial monitor to commands on a web page:

```WiFiServer server(80);

void setup() {
  //Initialize serial and wait for port to open:
  Serial.begin(9600);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  Serial.println("Access Point Web Server");

  pinMode(led, OUTPUT);

  // check for the WiFi module:
  if (WiFi.status() == WL_NO_SHIELD) {
    Serial.println("Communication with WiFi module failed!");
    // don't continue
    while (true);
  }



  Serial.print("Creating access point named: ");
  Serial.println(ssid);

 status = WiFi.beginAP(ssid, pass);
  if (status != WL_AP_LISTENING) {
    Serial.println("Creating access point failed");
    // don't continue
    while (true);
  }

  // wait 10 seconds for connection:
  delay(10000);

  // start the web server on port 80
  server.begin();

  // you're connected now, so print out the status
  printWifiStatus();

}

void loop() {
  // if there are incoming bytes available
  // from the server, read them and print them:
  // compare the previous status to the current status
  if (status != WiFi.status()) {
    // it has changed update the variable
    status = WiFi.status();

    if (status == WL_AP_CONNECTED) {
      // a device has connected to the AP
      Serial.println("Device connected to AP");
    } else {
      // a device has disconnected from the AP, and we are back in listening mode
      Serial.println("Device disconnected from AP");
    }
  }

  WiFiClient client = server.available();   // listen for incoming clients

  if (client) {                             // if you get a client,
    Serial.println("new client");           // print a message out the serial port
    String currentLine = "";                // make a String to hold incoming data from the client
    while (client.connected()) {            // loop while the client's connected
      if (client.available()) {             // if there's bytes to read from the client,
        char c = client.read();             // read a byte, then
        Serial.write(c);                    // print it out the serial monitor
        if (c == '\n') {                    // if the byte is a newline character

          // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:
          if (currentLine.length() == 0) {
            // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            // the content of the HTTP response follows the header:
            client.print("Clique aqui <a href=\"/H\">here</a> para ligar o LED<br>");
            client.print("Clique aqui <a href=\"/L\">here</a> para desligar o LED<br>");

            int randomReading = analogRead(A1);
            client.print("Leitura analogica: ");
            client.print(randomReading);

            // The HTTP response ends with another blank line:
            client.println();
            // break out of the while loop:
            break;
          }
          else {      // if you got a newline, then clear currentLine:
            currentLine = "";
          }
        }
        else if (c != '\r') {    // if you got anything else but a carriage return character,
          currentLine += c;      // add it to the end of the currentLine
        }

        // Check to see if the client request was "GET /H" or "GET /L":
        if (currentLine.endsWith("GET /H")) {
          digitalWrite(led, HIGH);               // GET /H turns the LED on
        }
        if (currentLine.endsWith("GET /L")) {
          digitalWrite(led, LOW);                // GET /L turns the LED off
        }
      }
    }
    // close the connection:
    client.stop();
    Serial.println("client disconnected");
  }

}

void printWifiStatus() {
  // print the SSID of the network you're attached to:
 Serial.print("SSID: ");
  Serial.println(WiFi.SSID());

  // print your WiFi shield's IP address:
  IPAddress ip = WiFi.localIP();
  Serial.print("IP Address: ");
  Serial.println(ip);

  // print where to go in a browser:
  Serial.print("To see this page in action, open a browser to http://");
  Serial.println(ip);
}
```

To look at the full code, go to programming at the top of the repository.
