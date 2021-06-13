# Sample Projects

- Install the "Arcadable" vscode extension.
- Open one of the sample directories (e.g. "Pong") with vscode, so that `arcadable.config.json` is in the root of the explorer in vscode.
- Press ctrl+shft+p, search for "Arcadable" and execute the command to start the emulator.


# arcadable.config.json
This is the base configuration for any Arcadable project.
The following structure is expected by the compiler:
```json
{
    "project": {
        "name": "4 Player Pong",
        "version": "1.0.0",
        "main": "/src/main.arc", 
        "export": "/out"
    },
    "system": {
        "screenWidth": 42,
        "screenHeight": 42,
        "mainsPerSecond": 300,
        "rendersPerSecond": 60,
        "digitalInputAmount": 16,
        "analogInputAmount": 8,
        "speakerOutputAmount": 4
    }
}
```
### `project.name`
Name of your project
### `project.version`
Version of your project
### `project.main`
Entry point for the compiler
### `project.export`
Output directory for binary exports
### `system.screenWidth` / `system.screenHeight`
Amount of pixels on the display
### `system.mainsPerSecond` / `system.rendersPerSecond`
Amount of times per second which the emulator will attempt to run the `main`/`render` function. Note: the c++ interpreter will attempt to run `main`/`render` as quickly as possible. Don't make the assumption in your code that `main`/`render` will be executed at the same frequency as this emulator.

The refresh rate (`render` per second) depends on the amount of pixels in the screen.
C++ implementation uses https://github.com/PaulStoffregen/WS2812Serial to send display data to the leds.

Every WS2812 led on a strip will add about 0.0317ms to the time it takes to refresh the display.

Luckily, the WS2812Serial library allows us to communicate with 7 led strips in parallel. So if we have a 42 by 42 display. We can split it up in 7 sections of 6 by 42.

So if the display is 42 by 42 pixels, we can get the following refresh rate:
```
1000/(0.0317 * 6 * 42) ≈ 125 frames per second
```
### `system.digitalInputAmount`
Amount of digital inputs (e.g. buttons)
### `system.analogInputAmount`
Amount of analog inputs (e.g. joysticks) 
### `system.speakerOutputAmount`
Amount of sound outputs

# Value types
All values are globally accessible. *There is no scope.*
### `Number`
Regular number value.

Truthy if value != 0.
```
varName: Number = 1;
varName: Number = 0.5;
```

### `Input`
Input values, will update automatically to mirror to user input.

Truthy if value != 0

The assigned value is the index of the button. E.g. `varName: Digital = 0;` will refer to the "first" button recognized by the system.

`Digital` type can have values 0 or 1.
`Analog` type can have a value between 0 and 1024.
```
varName: Analog = 1;
varName: Digital = 1;
```

### `Speaker`
Speaker values, used with the "tone" instruction to play sounds.
Assigning a new value to speaker directly is also possible.
In that case the new assigned value is the frequency the speaker should output. You will not have control over duration or volume like this; not recommended.

Use the `tone` instruction to play sounds.

The assigned value when initializing the value is the index of the speaker. E.g. `varName: Speaker = 0;` will refer to the "first" speaker recognized by the system.

```
varName: Speaker = 0;
```

### `Evaluation`
These values will be evaluated every time the value is used.
Unless it is a "static" evaluation, which will only be evaluated the first time this value is used.

Available evaluations/operators: `+, -, *, /, %, &, |, ^, <<, >>, pow, ==, !=, >, <, >=, <=`

Only "Numerical" values can be used in an evaluation. Values with type `List`, `String`, `Image` or `Data` cannot be used here.

Truthy if evaluation resolves to != 0
```
varName: Eval = 2 * otherVarName;
varName: Eval = static 2 * otherVarName;
```

### `Pixel`
Pixel value, represents the color value of the pixel at the defined location.

When set, the color at that position will change.

Reading this value will return the color of the pixel at the defined location.

Truthy if color at defined pixel position != 0
```
varName: Pixel = [5, 5];
varName: Pixel = [pixelPosXVar, pixelPosYVar];
```

### `Config`
System config value, represents system information.

Available values: `ScreenHeight, ScreenWidth, CurrentMillis, IsZigZag`

- CurrentMillis is the amount of millis since starting the game.
- IsZigZag is true if the leds on the display are arranged in a zigzag layout.

Truthy if value resolves to != 0
```
varName: Config = ScreenHeight;
```

### `Text`
Text value, thus far only used in `draw.drawText` instruction.

Truthy if length != 0
```
varName: String = "my text";
varName: String = 'my text';
```

### `Lists`
List value. Fixed size. Can be used with any data type, except for List<..> type (multidimensional arrays are not supported at this point).

Always truthy.
```
myList: List<Number> = [otherVarName, 2, 2.5];
```


List value pointer. Points to a specific value of a list.

Truthy if value at list position is truthy.
```
varName: ListValue = myList[1];
varName: ListValue = myList[otherVarName];
```

### `Images`
Image value. Represents image data.

The asset should be raw rgb values. In Gimp, export the image as "Raw Image Data", in the "Export as.." dialog file type selection.
Make sure the alpha channel is removed on all layers when exporting.

Image with/height/color are all numberical values.

```
varName: Image = ['assets/myImage.data', imageWidth, imageHeight, imageColor];

or

myImageData: Data = 'assets/myImage.data';
logo: Image = [myImageData, imageWidth, imageHeight, imageColor];
```

- `imageColor` in this initialization is the "key" color which should be transparent/ignored when rendering the image.


# Instruction types

## Drawing instruction types
Currently only decimal number types are supported. 
So defining color values might be unintuitive if you are used to hex notation for rgb values.
For now, just use tools like https://www.rapidtables.com/convert/number/hex-to-decimal.html To define your colors.
e.g. ff0000 (red) = 16711680
```
redColor: Number = 16711680;
```
### `Draw Line`
```
render: Function {
    draw.drawLine(color, x1, y1, x2, y2);
}
```

### `Draw Pixel`
```
color: Number = 16711680;
x: Number = 0;
y: Number = 0;
render: Function {
    draw.drawPixel(color, x, y);
}
```
Set the color of the pixel at position x, y.

Alternative:

```
color: Number = 16711680;
myPixel: Pixel = [0, 0];
render: Function {
    myPixel = color;
}
```

### `Draw/Fill circle`
```
render: Function {
    draw.drawCircle(color, radius, x, y);
    draw.fillCircle(color, radius, x, y);
}
```
### `Draw/Fill rectangle`
- `tl` is "top left"
- `br` is "bottom right"
```
render: Function {
    draw.drawRect(color, tlX, tlY, brX, brY);
    draw.fillRect(color, tlX, tlY, brX, brY);
}
```
### `Draw/Fill triangle`
```
render: Function {
    draw.drawTriangle(color, x1, y1, x2, y2, x3, y3);
    draw.fillTriangle(color, x1, y1, x2, y2, x3, y3);
}
```
### `Draw Text`
- Size 1 character is 6 pixels wide, 8 pixels tall
- Size 2 character is 12 pixels wide, 16 pixels tall
```
myText: String = "Hi!";
render: Function {
    draw.drawText(color, size, myText, x, y);
}
```
### `Draw Image`
See value type "Images" for more info.
```
myImageData: Data = 'logo.data';
logo: Image = [myImageData, 42, 42, 0];
orLikeThis: Image = ['logo.data', 42, 42, 0];

render: Function {
    draw.drawImage(0, 0, logo);
}
```

### `Set Rotation`
Set the rotation of the screen. 
Rotation can be an integer. Each increment of the integer will rotate the drawing context by 90 degrees.
Note: existing color values will not be rotated. To rotate existing values, clear the screen, rotate and redraw.
```
render: Function {
    draw.setRotation(rotation);
}
```
### `Clear`
Clears the entire screen. (all black)
```
render: Function {
    draw.clear;
}
```

## Other instruction types
### `Run instructionset/function`
Functions/methods can be defined like this:
```
setup: Function {

}
main: Function {

}

render: Function {

}
```

`setup`, `main` and `render` are the 3 default functions that must be present in any project.

Setup is executed once when starting the game.

Setup, main and render cannot be asynchronous.

Other functions can be defined in the same way.

```
myNormalFunction: Function {

}

myAsyncFunction: AsyncFunction {

}
```
Functions can be executed like this:
```
setup: Function {
    execute(myNormalFunction);
}
myNormalFunction: Function {

}

```

### `Debug log`
```
setup: Function {
    execute(myNormalFunction);
}
myNormalFunction: Function {
    log(123);
}
```
Expected output: 
```
123
```

### `async/await`
```
setup: Function {
    log(1);
    execute(myNormalFunction);
    execute(myAsyncFunction);
    log(4);
    execute(myNormalFunction);
}
myNormalFunction: Function {
    log(2);
}
myAsyncFunction: AsyncFunction {
    log(3);
    await execute(anotherAsyncFunction);
    log(6);
}
anotherAsyncFunction: AsyncFunction {
    wait(100);
    log(5);
}
```
Expected output: 
```
1
2
3
4
2
5
6
```

### `Play tone`
```
setup: Function {
    tone(mySpeaker, myVolume, myFrequency, myDuration); // start playing a tone and immediately continue code execution
}
myAsyncFunction: AsyncFunction {
    await tone(mySpeaker, myVolume, myFrequency, myDuration); // start playing a tone and wait until the tone is completed
}
```

### `Mutate value`
Available evaluations/operators: `+, -, *, /, %, &, |, ^, <<, >>, pow, ==, !=, >, <, >=, <=`

Some examples:

---
```
myNumber: Number = 0;
setup: Function {
    myNumber = myNumber + 1;
}
```
`myNumber` will equal 1.

---

```
myNumber: Number = 1;
setup: Function {
    myNumber = 1 == 2;
}
```
`myNumber` will equal 0.

---
```
myNumber: Number = 0;
myInput: Analog = 0;
setup: Function {
    myNumber = myInput;
}
```
`myNumber` will equal a value between 0 and 1024 determined by the current analog input value

---

### `Run condition`
`else if` currently not supported, just make an else if tower of doom.
```
setup: Function {
    if(myNumber > 0) {
        log(myNumber);
    } else {
        execute(myOtherFunction);
        if(myNumber < 0) {
            log(myNumber);
        } else {
            execute(myOtherFunction);
        }
    }
}
```

### `For loops`
Currently not supported. Although you can get identical behaviour like this:
```
setup: Function {
    execute(myLoop);
}

i: Number = 0;
myLoop: Function {
    if(i < 10) {
        log(i);
        i = i + 1;
        execute(myLoop);
    }
}
```