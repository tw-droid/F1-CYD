#include <FS.h>
#include "SPIFFS.h"
#include <FileFetcher.h>
#include <PNGdec.h>

#include <Arduino.h>
#include <WiFiClientSecure.h>


#include <ArduinoJson.h>

#include <WiFiMulti.h>
#include <HTTPClient.h>


WiFiMulti WiFiMulti;

WiFiClientSecure secured_client; //used for loading images

FileFetcher fileFetcher(secured_client); // used for storing images


// TFT screen (output)
#include <TFT_eSPI.h>
#define SCREEN_WIDTH 320
#define SCREEN_HEIGHT 240

TFT_eSPI tft = TFT_eSPI();
int tx = TFT_WHITE; // default text colour
int bg = TFT_BLACK; // default background_colour

int tz = 60*60*9.5; // number of seconds for the current timezone - this is +9.5 hours (South Australia)

// Install the "XPT2046_Touchscreen" library by Paul Stoffregen to use the Touchscreen - https://github.com/PaulStoffregen/XPT2046_Touchscreen
// Note: this library doesn't require further configuration
#include <XPT2046_Touchscreen.h>  

// Touchscreen pins
#define XPT2046_IRQ 36   // T_IRQ
#define XPT2046_MOSI 32  // T_DIN
#define XPT2046_MISO 39  // T_OUT
#define XPT2046_CLK 25   // T_CLK
#define XPT2046_CS 33    // T_CS

SPIClass touchscreenSPI = SPIClass(VSPI);
XPT2046_Touchscreen touchscreen(XPT2046_CS, XPT2046_IRQ);


// Touchscreen coordinates: (x, y) and pressure (z)
int x, y, z;


// Define a button object
TFT_eSPI_Button homebtn;   // main home button
TFT_eSPI_Button back;     // back button to go to previous  mode
TFT_eSPI_Button btns[10];  // array of buttons for data


// Google Root Certificate
const char *rootCACertificate = R"string_literal(
-----BEGIN CERTIFICATE-----
MIIFVzCCAz+gAwIBAgINAgPlk28xsBNJiGuiFzANBgkqhkiG9w0BAQwFADBHMQsw
CQYDVQQGEwJVUzEiMCAGA1UEChMZR29vZ2xlIFRydXN0IFNlcnZpY2VzIExMQzEU
MBIGA1UEAxMLR1RTIFJvb3QgUjEwHhcNMTYwNjIyMDAwMDAwWhcNMzYwNjIyMDAw
MDAwWjBHMQswCQYDVQQGEwJVUzEiMCAGA1UEChMZR29vZ2xlIFRydXN0IFNlcnZp
Y2VzIExMQzEUMBIGA1UEAxMLR1RTIFJvb3QgUjEwggIiMA0GCSqGSIb3DQEBAQUA
A4ICDwAwggIKAoICAQC2EQKLHuOhd5s73L+UPreVp0A8of2C+X0yBoJx9vaMf/vo
27xqLpeXo4xL+Sv2sfnOhB2x+cWX3u+58qPpvBKJXqeqUqv4IyfLpLGcY9vXmX7w
Cl7raKb0xlpHDU0QM+NOsROjyBhsS+z8CZDfnWQpJSMHobTSPS5g4M/SCYe7zUjw
TcLCeoiKu7rPWRnWr4+wB7CeMfGCwcDfLqZtbBkOtdh+JhpFAz2weaSUKK0Pfybl
qAj+lug8aJRT7oM6iCsVlgmy4HqMLnXWnOunVmSPlk9orj2XwoSPwLxAwAtcvfaH
szVsrBhQf4TgTM2S0yDpM7xSma8ytSmzJSq0SPly4cpk9+aCEI3oncKKiPo4Zor8
Y/kB+Xj9e1x3+naH+uzfsQ55lVe0vSbv1gHR6xYKu44LtcXFilWr06zqkUspzBmk
MiVOKvFlRNACzqrOSbTqn3yDsEB750Orp2yjj32JgfpMpf/VjsPOS+C12LOORc92
wO1AK/1TD7Cn1TsNsYqiA94xrcx36m97PtbfkSIS5r762DL8EGMUUXLeXdYWk70p
aDPvOmbsB4om3xPXV2V4J95eSRQAogB/mqghtqmxlbCluQ0WEdrHbEg8QOB+DVrN
VjzRlwW5y0vtOUucxD/SVRNuJLDWcfr0wbrM7Rv1/oFB2ACYPTrIrnqYNxgFlQID
AQABo0IwQDAOBgNVHQ8BAf8EBAMCAYYwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4E
FgQU5K8rJnEaK0gnhS9SZizv8IkTcT4wDQYJKoZIhvcNAQEMBQADggIBAJ+qQibb
C5u+/x6Wki4+omVKapi6Ist9wTrYggoGxval3sBOh2Z5ofmmWJyq+bXmYOfg6LEe
QkEzCzc9zolwFcq1JKjPa7XSQCGYzyI0zzvFIoTgxQ6KfF2I5DUkzps+GlQebtuy
h6f88/qBVRRiClmpIgUxPoLW7ttXNLwzldMXG+gnoot7TiYaelpkttGsN/H9oPM4
7HLwEXWdyzRSjeZ2axfG34arJ45JK3VmgRAhpuo+9K4l/3wV3s6MJT/KYnAK9y8J
ZgfIPxz88NtFMN9iiMG1D53Dn0reWVlHxYciNuaCp+0KueIHoI17eko8cdLiA6Ef
MgfdG+RCzgwARWGAtQsgWSl4vflVy2PFPEz0tv/bal8xa5meLMFrUKTX5hgUvYU/
Z6tGn6D/Qqc6f1zLXbBwHSs09dR2CQzreExZBfMzQsNhFRAbd03OIozUhfJFfbdT
6u9AWpQKXCBfTkBdYiJ23//OYb2MI3jSNwLgjt7RETeJ9r/tSQdirpLsQBqvFAnZ
0E6yove+7u7Y/9waLd64NnHi/Hm3lCXRSHNboTXns5lndcEZOitHTtNCjv0xyBZm
2tIMPNuzjsmhDYAPexZ3FL//2wmUspO8IFgV6dtxQ/PeEMMA3KgqlbbC1j+Qa3bb
bP6MvPJwNQzcmRk13NfIRmPVNnGuV/u3gm3c
-----END CERTIFICATE-----
)string_literal";


  struct tm timeinfo;

  struct driver {
    int id; // driver number
    String bname; //broadcast name
    String full_name; // full name
    String fname; // first name
    String lname; // last name;
    String team; // team name
    String colour; // team colour (24 bit RGB)
    String image; // url of headshot image
    String country; // country code
  };
  struct session {
    int meeting_id; // meeting id
    int session_id; // session id
    String loc; // location
    String session_name;
    String country; // country name
    String circuit; // circuit short name
    int yr; // year
  };
  struct session_result {
    int position;
    int id; //driver id
    int laps; //number of laps
    int points; // number of points
    float gap; // gap to leader
    int meeting_id; // meeting id
    int session_id; // session id
  };

  struct driver ds[30]; // new array of drivers
  struct session ss[10]; // new array of sessions
  struct session_result sr[100]; // new array of session results - id of array is meaningless

int driver_count = 0; // index of ds array to show where to put the next one...  
int session_count = 0; // index of ss array to show how many there are
int result_count = 0; // index of sr array to show how many there are


String mode; // mode - "allsessions" is to get all sessions in the last 7 days, "session" is for session details, "driver" for driver details
  int session_id; // variable to store selected session id
  int driver_id; // variable to store selected driver id

PNG png;
#define IMAGE_NAME "/img.png"

void setup() {
  // set up serial connection
  Serial.begin(115200);

    // Initialise SPIFFS filesystem - used to store the image when loaded
    bool spiffsInitSuccess = SPIFFS.begin(false) || SPIFFS.begin(true);
    if (!spiffsInitSuccess)
    {
        Serial.println("SPIFFS initialisation failed!");
        while (1)
            yield(); // Stay here twiddling thumbs waiting
    }

  // set up TFT display
  tft.init();
  tft.setRotation(1); // landscape mode
  tft.fillScreen(bg);
 
  // Start the SPI for the touchscreen and init the touchscreen
  touchscreenSPI.begin(XPT2046_CLK, XPT2046_MISO, XPT2046_MOSI, XPT2046_CS);
  touchscreen.begin(touchscreenSPI);
  // Set the Touchscreen rotation in landscape mode
  // Note: in some displays, the touchscreen might be upside down, so you might need to set the rotation to 3: touchscreen.setRotation(3);
  touchscreen.setRotation(1);
  
  
  loading("Connecting to Wifi");
  // connect to WIFI
   WiFi.mode(WIFI_STA);
WiFiMulti.addAP("AP", "PASSWORD");

  // wait for WiFi connection
  Serial.print("Waiting for WiFi to connect...");
  while ((WiFiMulti.run() != WL_CONNECTED)) {
    Serial.print(".");
  }
  Serial.println(" connected");
  updateDisplay();
  
  secured_client.setInsecure(); // You should probably use a cert for this!
  loading("Updating clock");
  setClock();
  updateDisplay();

  //Serial.println(getURL("https://api.jolpi.ca/ergast/"));
  //Serial.println(getURL("https://api.openf1.org/v1/sessions"));
  loading("Loading sessions");
  load_sessions();
  updateDisplay();
  // perhaps just get the drivers for session[0] to cache these at the start
 // loading("Caching drivers");
 // load_drivers(ss[0].session_id);

  // ready to go - start the mode
  mode = "allsessions";
  updateDisplay();

  
}

void loop() {
  bool done=false; // a boolean to ensure we only ever click on one button within the same loop
  
   // Check if the button is pressed
  if (touchscreen.tirqTouched() && touchscreen.touched()) {
   // Get Touchscreen points
    TS_Point p = touchscreen.getPoint();
    // Calibrate Touchscreen points with map function to the correct width and height
    x = map(p.x, 200, 3700, 1, SCREEN_WIDTH);
    y = map(p.y, 240, 3800, 1, SCREEN_HEIGHT);
    z = p.z;
      if (back.contains(x, y) && z>1000){
    // Button is pressed, do something
    Serial.println("BACK PRESSED");
    mode="session";
//    title = session_title;
    loading("Session");
    done=true;
    //done = true;
    //getURL();
    updateDisplay();
  }
  
    // activate home button
    if (homebtn.contains(x,y) && z>1000){ // z>1000 avoids unintentional touches 
      // button is pressed - do something
      Serial.println("HOME PRESSED");
      mode="allsessions";
      loading("Loading sessions");
      load_sessions(); // reload sessions
      done=true;     
      updateDisplay();
    }
    
 
for (int i=0; i<(sizeof(btns)/sizeof(btns[0])); i++) //sizeof array
{
  if(done==false && btns[i].contains(x,y) && z>1000)
  {
    done=true;
    if (mode=="allsessions")
    {
      Serial.println("Loading Session");
      mode="session";
      loading(ss[i].loc + " " + ss[i].session_name);
      session_id = i;
      load_session(ss[i].session_id);
      updateDisplay();
    }
    else if (mode=="session")
    {
      Serial.println("Loading Driver");
      mode="driver";
      driver_id = sr[i].id; //
      loading("Loading " + ds[driver_array(driver_id)].full_name);
      updateDisplay();
    }
  }
}

    }
    updateTimer();
  delay(10); // small delay in the main loop
}

void updateDisplay(){
  // main function for updating display
  tft.fillScreen(bg);
  tft.setTextWrap(true, false);
  tft.setTextPadding(5);
  tft.setTextColor(tx, bg);
  int x = 20;
  int y = 20;
  tft.setCursor(x,y);
  tft.setTextSize(2);
  if (mode==NULL)
  {
    tft.print("Welcome to F1 - please wait while the system initializes");
  }
  else
  {
    if (mode == "allsessions")
    {
       tft.print("All sessions"); // print title
       y+=30;
       
       for (int i=0; i<session_count; i++)
       {
         draw_button(&btns[i], ss[i].loc + " "+ ss[i].session_name, SCREEN_WIDTH/2, y, SCREEN_WIDTH-40, 20, TFT_WHITE, TFT_BLACK, TFT_WHITE, 1);
         y+=20;
       }
       
    }
    else if (mode== "session")
    {
      Serial.print("Result Count = ");
      Serial.println(result_count);
      tft.print(ss[session_id].loc + " " + ss[session_id].session_name); // print title
      y+=30;
      for (int i=0; (i<result_count && i<sizeof(btns)/sizeof(btns[0])); i++)
       {  Serial.println(driver_array(sr[i].id)); // need to load this before displaying it - in case it needs to load the new driver
         draw_button(&btns[i], ds[driver_array(sr[i].id)].full_name + " ("+sr[i].gap+")", SCREEN_WIDTH/2, y, SCREEN_WIDTH-40, 20, TFT_WHITE,  RGB2int(ds[driver_array(sr[i].id)].colour), TFT_WHITE, 1);
         y+=20;
       }
      
    }
    else if (mode=="driver")
    {
      tft.print(ds[driver_array(driver_id)].full_name); // print title
      y+=30;
      // display image
      if (driver_array(driver_id)!=-1)
      {
      Serial.print("Get Image: ");
      Serial.println((char *)ds[driver_array(driver_id)].image.c_str());
    int returnStatus = getImage((char *)ds[driver_array(driver_id)].image.c_str());
    if (returnStatus)    {        displayImage(IMAGE_NAME);    }
      }

    tft.setTextSize(2);
    tft.setCursor(x, y); 
    tft.setTextColor(RGB2int(ds[driver_array(driver_id)].colour), bg);
   
    tft.print("#");
    tft.print(driver_id);
    tft.println(" "+ds[driver_array(driver_id)].full_name);
    tft.println("Team: "+ds[driver_array(driver_id)].team);
    tft.println("Country: "+ds[driver_array(driver_id)].country);
    draw_button(&back, "BACK", 40, 215, 80, 25, TFT_WHITE, TFT_BLACK, TFT_WHITE, 2); // draw home button
    }
    draw_button(&homebtn, "HOME", 160, 215, 80, 25, TFT_WHITE, TFT_BLACK, TFT_WHITE, 2); // draw home button
 
    
  }
}
void draw_button(TFT_eSPI_Button* btn, String text, int xpos, int ypos, int w, int h, int txt_colour, int bg_colour, int border_colour, int font_size)
{
  btn->initButton(&tft,  // Pass the TFT object
                 xpos,  // x position
                 ypos,  // y position
                 w,  // width
                 h,   // height
                 txt_colour,  // text color
                 bg_colour,   // background color
                 border_colour,  // border color
                 "",  // button text - apply the long text below though
                 font_size);  // text font

  // Draw the button
  btn->drawButton(false, text); // write long text to the button
  return;
}

void updateTimer(){
  // update timer
  tft.setTextColor(tx, bg);
  tft.setTextSize(1);
  int x = 20;
  int y = 5;
  tft.setCursor(x, y); // Set the cursor
  int now = time(NULL);
  time_t nowt = time(NULL) + tz;
  gmtime_r(&nowt, &timeinfo);
  //Serial.print(F("Current time: "));
  //Serial.print(asctime(&timeinfo));
  tft.print(asctime(&timeinfo));
}

void loading(String str)
{
  tft.setTextColor(TFT_GREEN,bg);
  tft.setTextSize(1);
  tft.setCursor(20, SCREEN_HEIGHT/2);
  tft.print(str);
  tft.setTextColor(tx,bg);
}

void load_sessions(){
  char date[11];
  struct tm weekago;
  time_t prev_time = time(NULL) - 60*60*24*7;
  gmtime_r(&prev_time, &weekago);
  strftime(date, sizeof(date), "%Y-%m-%d", &weekago);
  Serial.println(date);
  
  String tmpurl = "https://api.openf1.org/v1/sessions?date_start>=";
  tmpurl += date;
  String data = getURL(tmpurl);
  Serial.println(data);

  const size_t CAPACITY = JSON_ARRAY_SIZE(30);
            StaticJsonDocument<CAPACITY> doc;
            deserializeJson(doc, data);
            //Serial.println(doc.as<String>());
            JsonArray array = doc.as<JsonArray>();
            int i=0;
            for(JsonVariant v: array){
              //Serial.println(v.as<String>());
              JsonDocument val;
              deserializeJson (val, v.as<String>());
              Serial.println("FOUND:");
              Serial.println(val.as<String>());
              ss[i] = { val["meeting_key"].as<int>(), val["session_key"].as<int>(), val["location"].as<String>(), val["session_name"].as<String>(), val["country_name"].as<String>(), val["circuit_short_name"].as<String>(), val["year"].as<int>()};
            i++;
            }
    session_count = i;
  return;
}

void load_session(int sid){
  // check if it is already in the array
  //sr has the sessions
  String tmpurl = "https://api.openf1.org/v1/session_result?session_key=";
  tmpurl += sid;
  Serial.println(tmpurl);
  String data = getURL(tmpurl);
  Serial.println(data);
   // do we blank out the session results array or just add to it???
   // currently just starting from 0 again (might leave results at the end)
   
  const size_t CAPACITY = JSON_ARRAY_SIZE(30);
            StaticJsonDocument<CAPACITY> doc;
            deserializeJson(doc, data);
            //Serial.println(doc.as<String>());
            JsonArray array = doc.as<JsonArray>();
            int i=0;
            for(JsonVariant v: array){
              //Serial.println(v.as<String>());
              JsonDocument val;
              deserializeJson (val, v.as<String>());
              Serial.println("FOUND:");
              Serial.println(val.as<String>());
              sr[i] = { val["position"].as<int>(), val["driver_number"].as<int>(), val["number_of_laps"].as<int>(), val["points"].as<int>(), val["gap_to_leader"].as<float>(), val["meeting_key"].as<int>(), val["session_key"].as<int>()};
            i++;
            }
  result_count = i;
  Serial.print("Finished loading session - total= ");
  Serial.println(result_count);
  return;
}

void load_drivers(int sid){

  // check if the driver already exists - ignore that, just load them for this session if this function is called
  // use the session id to limit the results
    String tmpurl = "https://api.openf1.org/v1/drivers?session_key=";
    tmpurl += sid;
    String data = getURL(tmpurl);
    const size_t CAPACITY = JSON_ARRAY_SIZE(30);
            StaticJsonDocument<CAPACITY> doc;
            deserializeJson(doc, data);
            //Serial.println(doc.as<String>());
            JsonArray array = doc.as<JsonArray>();
            int i=0;
            for(JsonVariant v: array){
              //Serial.println(v.as<String>());
              JsonDocument val;
              deserializeJson (val, v.as<String>());
              Serial.println("FOUND:");
              Serial.println(val.as<String>());
              if (val["headshot_url"]!="") // a little trick to only include if there is complete data
                {
                  ds[i] = { val["driver_number"].as<int>(), val["broadcast_name"].as<String>(), val["full_name"].as<String>(), val["first_name"].as<String>(), val["last_name"].as<String>(), val["team_name"].as<String>(), val["team_colour"].as<String>(), val["headshot_url"].as<String>(), val["country_code"].as<String>()};
                  i++;
                }
            }
  driver_count = i;
  Serial.print("Finished initial caching of drivers - total= ");
  Serial.println(driver_count);
  return;
}

int driver_array(int id) {
  // search the array for driver_id id
  for (int i=0; i<(sizeof(ds)/sizeof(ds[0])); i ++)
    {
      Serial.print("Checking Driver ");
      Serial.print(i);
      Serial.print(" with id ");
      Serial.print(ds[i].id);
      Serial.print(" named ");
      Serial.println(ds[i].full_name);
      
      if (ds[i].id == id) { return i;}
    }
       String tmpurl = "https://api.openf1.org/v1/drivers?driver_number=";
      tmpurl += id;
      if (session_id) { // if there is a current session, filter for just that
        tmpurl += "&session_key=";
        tmpurl += ss[session_id].session_id;
      }
    String data = getURL(tmpurl);
    const size_t CAPACITY = JSON_ARRAY_SIZE(30);
            StaticJsonDocument<CAPACITY> doc;
            deserializeJson(doc, data);
            //Serial.println(doc.as<String>());
            JsonArray array = doc.as<JsonArray>();
            int j=driver_count;
            bool found=false;
            for(JsonVariant v: array){
              //Serial.println(v.as<String>());
              if (found==false){
              JsonDocument val;
              deserializeJson (val, v.as<String>());
              Serial.println("FOUND:");
              Serial.println(val.as<String>());
              ds[j] = { val["driver_number"].as<int>(), val["broadcast_name"].as<String>(), val["full_name"].as<String>(), val["first_name"].as<String>(), val["last_name"].as<String>(), val["team_name"].as<String>(), val["team_colour"].as<String>(), val["headshot_url"].as<String>(), val["country_code"].as<String>()};
              found=true;
              j++;
              }
            }
  driver_count = j;
  Serial.print("Driver Count = ");
  Serial.println(driver_count);
  if (ds[driver_count].id == id){ // do a final check that it loaded correctly
    return driver_count;
  }
  else{   return -1; }
  
}

String getURL(String url) {
   NetworkClientSecure *client = new NetworkClientSecure;

  if (client) {
    client->setCACert(rootCACertificate);

    {
      // Add a scoping block for HTTPClient https to make sure it is destroyed before NetworkClientSecure *client is
      HTTPClient https;
https.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS); // test for following redirects
//YAY THIS WORKS - no longer need to try to setup the 302 redirection


      Serial.print("[HTTPS] begin...\n");
            
      if (https.begin(*client, url)) {  // HTTPS
        Serial.print("[HTTPS] GET... "+url+"\n");
        // start connection and send HTTP header
        //int httpCode = https.GET();
        int httpCode = https.sendRequest("GET");
        

        // httpCode will be negative on error
        if (httpCode > 0) {
          // HTTP header has been send and Server response header has been handled
          Serial.printf("[HTTPS] GET... code: %d\n", httpCode);

          // file found at server
          if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
            //String payload = https.getString();
            Stream& payloadStream = https.getStream();

            String payload = "";
            while (payloadStream.available()){
              char c = payloadStream.read();
              //Serial.prinpayloadStreamt(c);
              payload += c;
            }

            return payload;
            
          }
        } else {
          Serial.printf("[HTTPS] GET... failed, error: %s\n", https.errorToString(httpCode).c_str());
        }

        https.end();
      } else {
        Serial.printf("[HTTPS] Unable to connect\n");
      }

      // End extra scoping block
    }

    delete client;
  } else {
    Serial.println("Unable to create client");
  }
  return "";
}

void setClock() {

  configTime(0, 0, "pool.ntp.org");

  Serial.print(F("Waiting for NTP time sync: "));
  time_t nowSecs = time(nullptr);
  while (nowSecs < 8 * 3600 * 2) {
    delay(500);
    Serial.print(F("."));
    yield();
    nowSecs = time(nullptr);
  }

  Serial.println();

  gmtime_r(&nowSecs, &timeinfo);
  Serial.print(F("Current time: "));
  Serial.print(asctime(&timeinfo));

}

int RGB2int(String rgb) //convert a 24bit RGB colour hex code, into a 16bit TFT colour int
{
  int red, green, blue;
  red = (int)strtol(rgb.substring(0,2).c_str(), NULL, 16); 
  green = (int)strtol(rgb.substring(2,4).c_str(), NULL, 16);
  blue = (int)strtol(rgb.substring(4,6).c_str(), NULL, 16);
  
  long number = strtol((char*)rgb.c_str(), NULL, 16);
  Serial.println("Converted " + rgb + " to " + tft.color565(red, green, blue));
  return tft.color565(red, green, blue);
  //return (int)number;
}


// image functions below
void PNGDraw(PNGDRAW *pDraw)
{
    uint16_t usPixels[320];

    png.getLineAsRGB565(pDraw, usPixels, PNG_RGB565_BIG_ENDIAN, 0xffffffff);
    tft.pushImage(SCREEN_WIDTH-pDraw->iWidth, pDraw->y, pDraw->iWidth, 1, usPixels);
}

fs::File myfile;

void *myOpen(const char *filename, int32_t *size)
{
    myfile = SPIFFS.open(filename);
    *size = myfile.size();
    return &myfile;
}
void myClose(void *handle)
{
    if (myfile)
        myfile.close();
}
int32_t myRead(PNGFILE *handle, uint8_t *buffer, int32_t length)
{
    if (!myfile)
        return 0;
    return myfile.read(buffer, length);
}
int32_t mySeek(PNGFILE *handle, int32_t position)
{
    if (!myfile)
        return 0;
    return myfile.seek(position);
}

int displayImage(char *imageFileUri)
{
    //tft.fillScreen(TFT_BLACK);
    unsigned long lTime = millis();
    lTime = millis();
    Serial.println(imageFileUri);
    int rc = png.open((const char *)imageFileUri, myOpen, myClose, myRead, mySeek, PNGDraw);
    if (rc == PNG_SUCCESS)
    {
        Serial.printf("image specs: (%d x %d), %d bpp, pixel type: %d\n", png.getWidth(), png.getHeight(), png.getBpp(), png.getPixelType());
        rc = png.decode(NULL, 0);
        png.close();
    }
    else
    {
        Serial.print("error code: ");
        Serial.println(rc);
    }

    Serial.print("Time taken to decode and display Image (ms): ");
    Serial.println(millis() - lTime);

    return rc;
}

int getImage(char *imageUrl)
{
    // In this example I reuse the same filename
    // over and over
    if (SPIFFS.exists(IMAGE_NAME) == true)
    {
        Serial.println("Removing existing image");
        SPIFFS.remove(IMAGE_NAME);
    }

    fs::File f = SPIFFS.open(IMAGE_NAME, "w+");
    if (!f)
    {
        Serial.println("file open failed");
        return -1;
    }

    bool gotImage = fileFetcher.getFile(imageUrl, &f);

    // Make sure to close the file!
    f.close();

    return gotImage;
}
