# Practica_5
El objetivo de la practica es comprender el funcionamiento de los buses sistemas de comunicación entre periféricos; estos elementos pueden ser internos o externos al procesador y los buses pueden ser de tipo i2c, spi, i2s, usart. En esta en concreto trabajaremos el i2c.
## Codigo_5.1_Escáner
```cpp
#include <Arduino.h>
#include <Wire.h>
 
void setup()
{
Wire.begin();
Serial.begin(115200);
while (!Serial); // Leonardo: wait for serial monitor
Serial.println("\nI2C Scanner");
}
void loop()
{
byte error, address;
int nDevices;
Serial.println("Scanning...");
nDevices = 0;
for(address = 1; address < 127; address++ )
{
// The i2c_scanner uses the return value of
// the Write.endTransmisstion to see if
// a device did acknowledge to the address.
Wire.beginTransmission(address);
error = Wire.endTransmission();
if (error == 0)
{
Serial.print("I2C device found at address 0x");
 
if (address<16)
Serial.print("0");
Serial.print(address,HEX);
Serial.println(" !");
nDevices++;
}
else if (error==4)
{
Serial.print("Unknown error at address 0x");
if (address<16)
Serial.print("0");
Serial.println(address,HEX);
}
}
if (nDevices == 0)
Serial.println("No I2C devices found\n");
else
Serial.println("done\n");
delay(5000); // wait 5 seconds for next scan
}
```
## Funcionamiento
En la práctica hemos conocido los aspectos básicos de la comunicación i2C entre otras y este código nos permite ver si se ha establecido correctamente la conexión con un periférico que utiliza un bus i2C con nuestra ESP32.
En el código en el void loop() es dónde realizamos todos los procesos de comunicación i2c, el setup solo inicializamos la librería Wire para poder usar sus funciones. Primero declaramos variables que nos servirán para mostrar si la conexión es correcta y avisamos con un mensaje por pantalla que se va a empezar a escanear. Ahora hacemos un recorrido por todas las direcciones de la placa. Con ``` Wire.beginTransmission(address)``` inicializamos la transmisión con la dirección "adress" que pasamos como valor de entrada. La función ``` error = Wire.endTransmission();``` nos devuelve el estado de esta transmisión.
Si nos devuelve 0 entonces sabemos no se ha acabado la transmisión y por lo tanto se ha establecido correctamente. Después mostramos por pantalla la dirección de donde provienen los datos en hexadecimal y finalmente en el caso que nos devuelva un número diferente de 0 sería error.

## Codigo_5.2_DisplaySSD1306
```cpp
#include <Arduino.h>
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_I2CDevice.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
void testdrawline(void); // Draw many lines
void testdrawrect(void); // Draw rectangles (outlines)
void testfillrect(void); // Draw rectangles (filled)
void testdrawcircle(void ); // Draw circles (outlines)
void testfillcircle(void ); // Draw circles (filled)
void testdrawroundrect(void ); // Draw rounded rectangles (outlines)
void testfillroundrect(void); // Draw rounded rectangles (filled)
void testdrawtriangle(void); // Draw triangles (outlines)
void testfilltriangle(void); // Draw triangles (filled)
void testdrawchar(void); // Draw characters of the default font
void testdrawstyles(void); // Draw 'stylized' characters
void testscrolltext(void); // Draw scrolling text
void testdrawbitmap(void); // Draw a small bitmap image
void testanimate(const uint8_t *bitmap, uint8_t w, uint8_t h);
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 32 // OLED display height, in pixels
// Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
// The pins for I2C are defined by the Wire-library.
// On an arduino UNO: A4(SDA), A5(SCL)
// On an arduino MEGA 2560: 20(SDA), 21(SCL)
// On an arduino LEONARDO: 2(SDA), 3(SCL), ...
#define OLED_RESET -1 // Reset pin # (or -1 if sharing Arduino reset pin)
#define SCREEN_ADDRESS 0x3C ///< See datasheet for Address; 0x3D for 128x64, 0x3C for 128x32
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
#define NUMFLAKES 10 // Number of snowflakes in the animation example
#define LOGO_HEIGHT 16
#define LOGO_WIDTH 16
static const unsigned char PROGMEM logo_bmp[] =
{ B00000000, B11000000,
B00000001, B11000000,
B00000001, B11000000,
B00000011, B11100000,
B11110011, B11100000,
B11111110, B11111000,
B01111110, B11111111,
B00110011, B10011111,
B00011111, B11111100,
B00001101, B01110000,
B00011011, B10100000,
B00111111, B11100000,
B00111111, B11110000,
B01111100, B11110000,
B01110000, B01110000,
B00000000, B00110000 };
void setup() {
Serial.begin(115200);
// SSD1306_SWITCHCAPVCC = generate display voltage from 3.3V internally
if(!display.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS)) {
Serial.println(F("SSD1306 allocation failed"));
for(;;); // Don't proceed, loop forever
}
// Show initial display buffer contents on the screen --
// the library initializes this with an Adafruit splash screen.
display.display();
delay(2000); // Pause for 2 seconds
// Clear the buffer
display.clearDisplay();
// Draw a single pixel in white
display.drawPixel(10, 10, SSD1306_WHITE);
// Show the display buffer on the screen. You MUST call display() after
// drawing commands to make them visible on screen!
display.display();
delay(2000);
// display.display() is NOT necessary after every single drawing command,
// unless that's what you want...rather, you can batch up a bunch of
// drawing operations and then update the screen all at once by calling
// display.display(). These examples demonstrate both approaches...
testdrawline(); // Draw many lines
testdrawrect(); // Draw rectangles (outlines)
testfillrect(); // Draw rectangles (filled)
testdrawcircle(); // Draw circles (outlines)
testfillcircle(); // Draw circles (filled)
testdrawroundrect(); // Draw rounded rectangles (outlines)
testfillroundrect(); // Draw rounded rectangles (filled)
testdrawtriangle(); // Draw triangles (outlines)
testfilltriangle(); // Draw triangles (filled)
testdrawchar(); // Draw characters of the default font
testdrawstyles(); // Draw 'stylized' characters
testscrolltext(); // Draw scrolling text
testdrawbitmap(); // Draw a small bitmap image
// Invert and restore display, pausing in-between
display.invertDisplay(true);
delay(1000);
display.invertDisplay(false);
delay(1000);
testanimate(logo_bmp, LOGO_WIDTH, LOGO_HEIGHT); // Animate bitmaps
}
void loop() {
}
void testdrawline() {
int16_t i;
display.clearDisplay(); // Clear display buffer
for(i=0; i<display.width(); i+=4) {
display.drawLine(0, 0, i, display.height()-1, SSD1306_WHITE);
display.display(); // Update screen with each newly-drawn line
delay(1);
}
for(i=0; i<display.height(); i+=4) {
display.drawLine(0, 0, display.width()-1, i, SSD1306_WHITE);
display.display();
delay(1);
}
delay(250);
display.clearDisplay();
for(i=0; i<display.width(); i+=4) {
display.drawLine(0, display.height()-1, i, 0, SSD1306_WHITE);
display.display();
delay(1);
}
for(i=display.height()-1; i>=0; i-=4) {
display.drawLine(0, display.height()-1, display.width()-1, i, SSD1306_WHITE);
display.display();
delay(1);
}
delay(250);
display.clearDisplay();
for(i=display.width()-1; i>=0; i-=4) {
display.drawLine(display.width()-1, display.height()-1, i, 0, SSD1306_WHITE);
display.display();
delay(1);
}
for(i=display.height()-1; i>=0; i-=4) {
display.drawLine(display.width()-1, display.height()-1, 0, i, SSD1306_WHITE);
display.display();
delay(1);
}
delay(250);
display.clearDisplay();
for(i=0; i<display.height(); i+=4) {
display.drawLine(display.width()-1, 0, 0, i, SSD1306_WHITE);
display.display();
delay(1);
}
for(i=0; i<display.width(); i+=4) {
display.drawLine(display.width()-1, 0, i, display.height()-1, SSD1306_WHITE);
display.display();
delay(1);
}
delay(2000); // Pause for 2 seconds
}
void testdrawrect(void) {
display.clearDisplay();
for(int16_t i=0; i<display.height()/2; i+=2) {
display.drawRect(i, i, display.width()-2*i, display.height()-2*i, SSD1306_WHITE);
display.display(); // Update screen with each newly-drawn rectangle
delay(1);
}
delay(2000);
}
void testfillrect(void) {
display.clearDisplay();
for(int16_t i=0; i<display.height()/2; i+=3) {
// The INVERSE color is used so rectangles alternate white/black
display.fillRect(i, i, display.width()-i*2, display.height()-i*2, SSD1306_INVERSE);
display.display(); // Update screen with each newly-drawn rectangle
delay(1);
}
delay(2000);
}
void testdrawcircle(void) {
display.clearDisplay();
for(int16_t i=0; i<max(display.width(),display.height())/2; i+=2) {
display.drawCircle(display.width()/2, display.height()/2, i, SSD1306_WHITE);
display.display();
delay(1);
}
delay(2000);
}
void testfillcircle(void) {
display.clearDisplay();
for(int16_t i=max(display.width(),display.height())/2; i>0; i-=3) {
// The INVERSE color is used so circles alternate white/black
display.fillCircle(display.width() / 2, display.height() / 2, i, SSD1306_INVERSE);
display.display(); // Update screen with each newly-drawn circle
delay(1);
}
delay(2000);
}
void testdrawroundrect(void) {
display.clearDisplay();
for(int16_t i=0; i<display.height()/2-2; i+=2) {
display.drawRoundRect(i, i, display.width()-2*i, display.height()-2*i,
display.height()/4, SSD1306_WHITE);
display.display();
delay(1);
}
delay(2000);
}
void testfillroundrect(void) {
display.clearDisplay();
for(int16_t i=0; i<display.height()/2-2; i+=2) {
// The INVERSE color is used so round-rects alternate white/black
display.fillRoundRect(i, i, display.width()-2*i, display.height()-2*i,
display.height()/4, SSD1306_INVERSE);
display.display();
delay(1);
}
delay(2000);
}
void testdrawtriangle(void) {
display.clearDisplay();
for(int16_t i=0; i<max(display.width(),display.height())/2; i+=5) {
display.drawTriangle(
display.width()/2 , display.height()/2-i,
display.width()/2-i, display.height()/2+i,
display.width()/2+i, display.height()/2+i, SSD1306_WHITE);
display.display();
delay(1);
}
delay(2000);
}
void testfilltriangle(void) {
display.clearDisplay();
for(int16_t i=max(display.width(),display.height())/2; i>0; i-=5) {
// The INVERSE color is used so triangles alternate white/black
display.fillTriangle(
display.width()/2 , display.height()/2-i,
display.width()/2-i, display.height()/2+i,
display.width()/2+i, display.height()/2+i, SSD1306_INVERSE);
display.display();
delay(1);
}
delay(2000);
}
void testdrawchar(void) {
display.clearDisplay();
display.setTextSize(1); // Normal 1:1 pixel scale
display.setTextColor(SSD1306_WHITE); // Draw white text
display.setCursor(0, 0); // Start at top-left corner
display.cp437(true); // Use full 256 char 'Code Page 437' font
// Not all the characters will fit on the display. This is normal.
// Library will draw what it can and the rest will be clipped.
for(int16_t i=0; i<256; i++) {
if(i == '\n') display.write(' ');
else display.write(i);
}
display.display();
delay(2000);
}
void testdrawstyles(void) {
display.clearDisplay();
display.setTextSize(1); // Normal 1:1 pixel scale
display.setTextColor(SSD1306_WHITE); // Draw white text
display.setCursor(0,0); // Start at top-left corner
display.println(F("Hello, world!"));
display.setTextColor(SSD1306_BLACK, SSD1306_WHITE); // Draw 'inverse' text
display.println(3.141592);
display.setTextSize(2); // Draw 2X-scale text
display.setTextColor(SSD1306_WHITE);
display.print(F("0x")); display.println(0xDEADBEEF, HEX);
display.display();
delay(2000);
}
void testscrolltext(void) {
display.clearDisplay();
display.setTextSize(2); // Draw 2X-scale text
display.setTextColor(SSD1306_WHITE);
display.setCursor(10, 0);
display.println(F("scroll"));
display.display(); // Show initial text
delay(100);
// Scroll in various directions, pausing in-between:
display.startscrollright(0x00, 0x0F);
delay(2000);
display.stopscroll();
delay(1000);
display.startscrollleft(0x00, 0x0F);
delay(2000);
display.stopscroll();
delay(1000);
display.startscrolldiagright(0x00, 0x07);
delay(2000);
display.startscrolldiagleft(0x00, 0x07);
delay(2000);
display.stopscroll();
delay(1000);
}
void testdrawbitmap(void) {
display.clearDisplay();
display.drawBitmap(
(display.width() - LOGO_WIDTH ) / 2,
(display.height() - LOGO_HEIGHT) / 2,
logo_bmp, LOGO_WIDTH, LOGO_HEIGHT, 1);
display.display();
delay(1000);
}
#define XPOS 0 // Indexes into the 'icons' array in function below
#define YPOS 1
#define DELTAY 2
void testanimate(const uint8_t *bitmap, uint8_t w, uint8_t h) {
int8_t f, icons[NUMFLAKES][3];
// Initialize 'snowflake' positions
for(f=0; f< NUMFLAKES; f++) {
icons[f][XPOS] = random(1 - LOGO_WIDTH, display.width());
icons[f][YPOS] = -LOGO_HEIGHT;
icons[f][DELTAY] = random(1, 6);
Serial.print(F("x: "));
Serial.print(icons[f][XPOS], DEC);
Serial.print(F(" y: "));
Serial.print(icons[f][YPOS], DEC);
Serial.print(F(" dy: "));
Serial.println(icons[f][DELTAY], DEC);
}
for(;;) { // Loop forever...
display.clearDisplay(); // Clear the display buffer
// Draw each snowflake:
for(f=0; f< NUMFLAKES; f++) {
display.drawBitmap(icons[f][XPOS], icons[f][YPOS], bitmap, w, h, SSD1306_WHITE);
}
display.display(); // Show the display buffer on the screen
delay(200); // Pause for 1/10 second
// Then update coordinates of each flake...
for(f=0; f< NUMFLAKES; f++) {
icons[f][YPOS] += icons[f][DELTAY];
// If snowflake is off the bottom of the screen...
if (icons[f][YPOS] >= display.height()) {
// Reinitialize to a random position, just off the top
icons[f][XPOS] = random(1 - LOGO_WIDTH, display.width());
icons[f][YPOS] = -LOGO_HEIGHT;
icons[f][DELTAY] = random(1, 6);
}
}
}
}
```
## Funcionamiento
Utilizamos el código proporcionado por la práctica para OLEDs monocromo basados en los drivers SSD1306. Simplemente es un ejemplo para comunicarnos usando I2C con un display. Respecto al código como tal, antes del setup se realiza toda la configuración y definiciones necesarias  para el programa como las diferentes funciones de las formas que se dibujan, las dimensiones de la pantalla del display, etc. Después, en el setup inicializamos el puerto serie y el objeto del display con display.display(). Comprobamos que funcione todo correctamente dibujando un pequeño pixel en pantalla para despues limpiarla. Finalmente en el loop se programan todas las funciones void de dibujo y se van ejecutando a medida que limpiamos la pantalla para poder mostrar la siguiente.
