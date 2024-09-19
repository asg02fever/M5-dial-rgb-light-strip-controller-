#include <FastLED.h>
#include "M5Dial.h"

// LED strip settings
#define LED_PIN 13
#define LED_COUNT 30
#define LED_TYPE SK6812
#define COLOR_ORDER GRB

CRGB leds[LED_COUNT];
int activeLEDs = LED_COUNT;

int brightness = 50;
int currentOption = 0;
long oldPosition = 0;
long encoderPosition = 0;
int mainMenuScrollThreshold = 5;  // Higher threshold for main menu scrolling
int colorMenuScrollThreshold = 1; // Lower threshold for faster color menu scrolling
bool optionChanged = false;
bool ledState = false;

// Menu state variables
bool inSubMenu = false;
int colorOptionIndex = 0;
bool inLEDCountMenu = false;
bool inBrightnessMenu = false;

// Function to draw a smoother color gradient circle for the picker
void drawColorGradientPicker(LovyanGFX *gfx) {
    gfx->fillScreen(TFT_BLACK);

    int centerX = gfx->width() / 2;
    int centerY = gfx->height() / 2;
    int outerRadius = gfx->width() / 2.5; // Wider gradient circle
    int innerRadius = outerRadius - 30;   // Inner radius for the gradient ring

    // Draw color gradient with more steps to smooth the transitions
    for (int i = 0; i < 360; i++) {
        float angle = radians(i - 90);
        int outerX = centerX + cos(angle) * outerRadius;
        int outerY = centerY + sin(angle) * outerRadius;
        int innerX = centerX + cos(angle) * innerRadius;
        int innerY = centerY + sin(angle) * innerRadius;

        // Higher resolution by calculating intermediate values for smoother rendering
        for (int j = 0; j < 3; j++) { // Increase iterations for finer steps
            float blendFactor = j / 3.0;
            int blendedOuterX = innerX + (outerX - innerX) * blendFactor;
            int blendedOuterY = innerY + (outerY - innerY) * blendFactor;

            CRGB rgbColor = CHSV(i, 255, 255);
            uint16_t color = gfx->color565(rgbColor.r, rgbColor.g, rgbColor.b);
            gfx->drawPixel(blendedOuterX, blendedOuterY, color);
        }
    }

    // Draw the cursor outside the gradient
    float cursorAngle = radians(colorOptionIndex - 90);
    int cursorX = centerX + cos(cursorAngle) * (outerRadius + 15);
    int cursorY = centerY + sin(cursorAngle) * (outerRadius + 15);
    gfx->fillCircle(cursorX, cursorY, 8, TFT_WHITE);
}

void drawMenu(LovyanGFX *gfx) {
    gfx->fillScreen(TFT_BLACK);

    int centerX = gfx->width() / 2;
    int centerY = gfx->height() / 2;
    int radius = gfx->width() / 2.5;
    int circleRadius = 25;
    int glowRadius = circleRadius + 10;

    if (inSubMenu) {
        drawColorGradientPicker(gfx);
    } else if (inLEDCountMenu) {
        gfx->setTextColor(TFT_WHITE);
        gfx->setTextSize(3);
        gfx->setCursor(gfx->width() / 2 - 50, gfx->height() / 2 - 20);
        gfx->print("LEDs: ");
        gfx->print(activeLEDs);
    } else if (inBrightnessMenu) {
        gfx->setTextColor(TFT_WHITE);
        gfx->setTextSize(3);
        gfx->setCursor(gfx->width() / 2 - 80, gfx->height() / 2 - 20);
        gfx->print("Brightness: ");
        gfx->print(brightness);
    } else {
        // Draw main menu with icons
        float angleStep = 360.0 / 4;

        for (int i = 0; i < 4; i++) {
            float angle = radians(angleStep * i - 90);
            int x = centerX + cos(angle) * (radius - 10);
            int y = centerY + sin(angle) * (radius - 10);

            if (i == currentOption) {
                gfx->fillCircle(x, y, glowRadius, TFT_ORANGE);
            } else {
                gfx->fillCircle(x, y, glowRadius, TFT_BLACK);
            }

            gfx->fillCircle(x, y, circleRadius, TFT_WHITE);

            // Draw icons
            gfx->setTextColor(TFT_BLACK);
            gfx->setTextSize(2);
            switch (i) {
            case 0: // Toggle LED
                gfx->setCursor(x - 10, y - 10);
                gfx->print("on/off"); // Lightbulb icon
                break;
            case 1: // Color Picker
                gfx->setCursor(x - 10, y - 10);
                gfx->print("C"); // Painter brush icon
                break;
            case 2: // Brightness
                gfx->setCursor(x - 10, y - 10);
                gfx->print(" %"); // Percentage icon
                break;
            case 3: // LED Count
                gfx->setCursor(x - 10, y - 10);
                gfx->print("+/-"); // Plus/Plus icon
                break;
            }
        }
    }
}

void handleSelection(LovyanGFX *gfx) {
    if (inSubMenu) {
        // Apply selected color
        CRGB color = CHSV(colorOptionIndex, 255, 255);
        fill_solid(leds, activeLEDs, color);
        FastLED.setBrightness(brightness);
        FastLED.show();

        inSubMenu = false;
        drawMenu(gfx);
    } else if (inLEDCountMenu) {
        inLEDCountMenu = false;
        drawMenu(gfx);
    } else if (inBrightnessMenu) {
        inBrightnessMenu = false;
        drawMenu(gfx);
    } else {
        switch (currentOption) {
        case 0:
            ledState = !ledState;
            if (ledState) {
                fill_solid(leds, activeLEDs, CRGB::White);
                FastLED.setBrightness(brightness);
            } else {
                fill_solid(leds, LED_COUNT, CRGB::Black);
            }
            FastLED.show();
            break;

        case 1:
            inSubMenu = true;
            drawMenu(gfx);
            break;

        case 2:
            inBrightnessMenu = true;
            drawMenu(gfx);
            break;

        case 3:
            inLEDCountMenu = true;
            drawMenu(gfx);
            break;
        }
    }
}

void setup() {
    auto cfg = M5.config();
    M5Dial.begin(cfg, true, false);

    FastLED.addLeds<LED_TYPE, LED_PIN, COLOR_ORDER>(leds, LED_COUNT);
    FastLED.clear();
    FastLED.setBrightness(0);
    FastLED.show();

    drawMenu(&M5Dial.Display);
}

void loop() {
    M5Dial.update();

    encoderPosition = M5Dial.Encoder.read();

    // Adjust scrolling sensitivity based on the current menu state
    int threshold = inSubMenu ? colorMenuScrollThreshold : mainMenuScrollThreshold;

    if (abs(encoderPosition - oldPosition) >= threshold) {
        if (encoderPosition > oldPosition) {
            if (inSubMenu) {
                colorOptionIndex = (colorOptionIndex + 5) % 360; // Faster scrolling for color picker
            } else if (inLEDCountMenu) {
                activeLEDs = constrain(activeLEDs + 1, 1, LED_COUNT);
            } else if (inBrightnessMenu) {
                brightness = constrain(brightness + 10, 0, 255);
            } else {
                currentOption = (currentOption + 1) % 4;
            }
        } else {
            if (inSubMenu) {
                colorOptionIndex = (colorOptionIndex - 5 + 360) % 360; // Faster scrolling for color picker
            } else if (inLEDCountMenu) {
                activeLEDs = constrain(activeLEDs - 1, 1, LED_COUNT);
            } else if (inBrightnessMenu) {
                brightness = constrain(brightness - 10, 0, 255);
            } else {
                currentOption = (currentOption - 1 + 4) % 4;
            }
        }

        oldPosition = encoderPosition;
        drawMenu(&M5Dial.Display);
    }

    // Handle BtnA press
    if (M5Dial.BtnA.wasPressed()) {
        handleSelection(&M5Dial.Display);
    }
}
