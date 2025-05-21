<!--
author: André Dietrich

email: LiaScript@web.de

comment: Embed the MicroPython simulator for the BBC MicroBit into a LiaScript course.

version: 0.0.1

edit:  true

link:  style.css

@onload
window.createSensorUI = function (simulator, selector, state) {
  const parent = document.querySelector(selector)
  parent.classList.add('sensor-ui')
  parent.innerHTML = ''

  function onChange(id, value) {
    simulator.postMessage({ kind: 'set_value', id, value }, '*')
  }

  function makeCard(id) {
    const card = document.createElement('div')
    card.className = 'sensor-card'
    const label = document.createElement('span')
    label.innerText = id
    card.appendChild(label)
    return card
  }

  for (const sensor of Object.values(state)) {
    const { type, id } = sensor
    const card = makeCard(id)

    if (type === 'range') {
      const input = document.createElement('input')
      input.type = 'range'
      input.min = sensor.min
      input.max = sensor.max
      input.value = sensor.value
      input.addEventListener('input', (e) => onChange(id, e.target.value))
      card.appendChild(input)
    } else if (type === 'enum') {
      const select = document.createElement('select')
      for (const choice of sensor.choices) {
        const opt = new Option(
          choice,
          choice,
          choice === sensor.value,
          choice === sensor.value
        )
        select.appendChild(opt)
      }
      select.addEventListener('change', (e) => onChange(id, e.target.value))
      card.appendChild(select)
    } else {
      const info = { ...sensor }
      delete info.type
      const infoCode = document.createElement('code')
      infoCode.innerText = JSON.stringify(info)
      card.appendChild(infoCode)
    }

    parent.appendChild(card)
  }
}

@end


@microbit: @microbit_(@uid,```@input```,false)

@microbit.withSensorUI: @microbit_(@uid,```@input```,true)

@microbit_
<script>
const iframe = document.querySelector('#simulator_@0')
const simulator = iframe.contentWindow
let state = {}

const onMessage = (e) => {
    if (e.source !== simulator) return;
    const { data } = e;
    switch (data.kind) {
      case 'serial_output':
        console.stream(data.data);
        break;
      case 'log_output':
        if (data.headings) console.log(data.headings);
        if (data.data)     console.log(data.data);
        break;
      case 'radio_output':
        const text = new TextDecoder()
          .decode(e.data.data)
          .replace(/^\x01\x00\x01/, '')
        console.log(text)
        break
      case 'ready':
        state = data.state
        window.console.log('ready', data.state);
        if(@2) {
          createSensorUI(simulator, "#sensors_@0", state)
        }
        break
      case 'state_change':
        state = {
          ...state,
          ...data.change,
        }
        if(@2) {
          createSensorUI(simulator, "#sensors_@0", state)
        }
        break
      case 'log_delete':
        console.debug('[log_delete]');
        break;
      case 'request_flash':
        simulator.postMessage({
          kind: 'flash',
          filesystem: {
            'main.py': new TextEncoder().encode(`@1`),
          },
        }, '*');
        simulator.postMessage({
          kind: 'reset',
        }, '*');
        send.lia("LIA: terminal");
        break;
      case 'internal_error':
        console.error('Simulator internal error:', data.error);
        break;
      default:
        console.warn('Unknown message kind:', data.kind);
        break;
    }
  };

window.addEventListener('message', onMessage);

send.handle("input", (data) => {
  simulator.postMessage({
      kind: 'serial_input',
      data: data + "\r\n",
    },
    '*'
  )
})

send.handle("stop", () => {
    // detach the message listener
    window.removeEventListener('message', onMessage);

    // reset the iframe by reloading its src
    // (you can also set src="" then back, or completely remove/add the element)
    iframe.src = iframe.src;

    // optionally hide it if you want:
    iframe.style.display = "none";

    document.getElementById("sensors_@0").innerHTML = "";
  });

iframe.style.display = "block"

"LIA: wait"
</script>

<iframe
  id="simulator_@0"
  src="https://python-simulator.usermbit.org/v/0.1/simulator.html?color=green"
  title="Simulator"
  frameborder="0"
  scrolling="no"
  sandbox="allow-scripts allow-same-origin"
  style="width: 100%; max-width: 500px; aspect-ratio: 10 / 8; display: none"
></iframe>

<div id="sensors_@0" class="sensor-ui"></div>
@end

-->

# MicroBit-Simulator


                          --{{0}}--
The MicroBit-Simulator allows you to run MicroPython code in a browser for the simulator of the BBC MicroBit.


__Try it on LiaScript:__

https://liascript.github.io/course/?https://raw.githubusercontent.com/liaTemplates/MicroBit-Simulator/main/README.md

__See the project on Github:__

https://github.com/liaTemplates/MicroBit-Simulator

                         --{{1}}--
Like with other LiaScript templates, there are three ways to integrate
WebSerial, but the easiest way is to copy the import statement into your project.
See the implementation in [Sec. Implementation](#implementation).

                           {{1}}
1. Load the latest macros via (this might cause breaking changes)

   `import: https://raw.githubusercontent.com/liaTemplates/MicroBit-Simulator/main/README.md`

   or the current version 0.0.1 via:

   `import: https://raw.githubusercontent.com/LiaTemplates/MicroBit-Simulator/0.0.1/README.md`

2. __Copy the definitions into your Project__

3. Clone this repository on GitHub

                          --{{2}}--
Additionally it is recommended to set the `persistent` option to `true` in the
LiaScript course. This will will prevent the simulator from being destroyed if a new slide is loaded.

    {{2}}
```` markdown
<!--
...
persistent: true
-->

# title ...
````


## `@microbit`

                     --{{0}}--
The `@microbit` macro attached to a LiaScript code-block will instantiate a simulator, whereby the user has to activate the simulator at first use. It is possible to touch the buttons or to change the display...

```` markdown
``` python
from microbit import *

while True:
    if button_a.was_pressed():
        print("A")
        display.show('A')
    if button_b.was_pressed():
        print("B")
        display.show('B')
```
@microbit
````

__Result:__


Buttons

``` python
from microbit import *

while True:
    if button_a.was_pressed():
        print("A")
        display.show('A')
    if button_b.was_pressed():
        print("B")
        display.show('B')
```
@microbit


## `@microbit.withSensorUI`

                     --{{0}}--
The `@microbit.withSensorUI` macro attached to a LiaScript code-block will instantiate a simulator with a sensor UI. The user can change the values of the sensors in the simulator.

```` markdown
``` python
from microbit import *

while True:
    print(display.read_light_level())
    sleep(1000);
```
@microbit.withSensorUI
````

__Result:__

``` python
from microbit import *

while True:
    print(display.read_light_level())
    sleep(1000);
```
@microbit.withSensorUI


## Examples

### Accelerometer

``` python
from microbit import *

last = None
while True:
    current = accelerometer.current_gesture()
    if current != last:
        last = current
        print(current)
```
@microbit.withSensorUI


### Audio


``` python
# micro:bit demo, playing a sine wave using AudioFrame's
# This doesn't sound great in the sim or on V2.
# The time per frame is significantly higher in the sim.

import math
import audio
from microbit import pin0, running_time

print("frames should take {} ms to play".format(32 / 7812.5 * 1000))


def repeated_frame(frame, count):
    for i in range(count):
        yield frame


frame = audio.AudioFrame()
nframes = 100

for freq in range(2, 17, 2):
    l = len(frame)
    for i in range(l):
        frame[i] = int(127 * (math.sin(math.pi * i / l * freq) + 1))
    print("play tone at frequency {}".format(freq * 7812.5 / 32))
    t0 = running_time()
    audio.play(repeated_frame(frame, nframes), wait=True, pin=pin0)
    dt_ms = running_time() - t0
    print("{} frames took {} ms = {} ms/frame".format(nframes, dt_ms, dt_ms / nframes))

```
@microbit


### Background music and display


``` python
from microbit import *
import music

music.play(['f', 'a', 'c', 'e'], wait=False)
display.scroll("Music and scrolling in the background", wait=False)
print("This should be printed before the scroll and play complete")
```
@microbit


### Compass

``` python
# Imports go at the top
from microbit import *

# Code in a 'while True:' loop repeats forever
while True:
    print("x: ", compass.get_x())
    print("y: ", compass.get_y())
    print("z: ", compass.get_z())
    print("heading: ", compass.heading())
    print("field strength: ", compass.get_field_strength())
    sleep(1000)
```
@microbit.withSensorUI

### Data logging

``` python
# Imports go at the top
from microbit import *
import log


# Code in a 'while True:' loop repeats forever
while True:
    log.set_labels('temperature', 'sound', 'light')
    log.add({
      'temperature': temperature(),
      'sound': microphone.sound_level(),
      'light': display.read_light_level()
    })
    sleep(1000)
```
@microbit.withSensorUI

### Display

``` python
from microbit import *

display.scroll("Hello")
display.scroll("World", wait=False)
for _ in range(0, 50):
    print(display.get_pixel(2, 2))
    sleep(100)
display.show(Image.HEART)
sleep(1000)
display.clear()
```
@microbit

### Inline assembler

``` python
from microbit import *

# Unsupported in the simulator
@micropython.asm_thumb
def asm_add(r0, r1):
    add(r0, r0, r1)

while True:
    if button_a.was_pressed():
        print(1 + 2)
    elif button_b.was_pressed():
        print(asm_add(1, 2))
```
@microbit

### Microphone

``` python
from microbit import *

while True:
    print(microphone.sound_level())
    if microphone.was_event(SoundEvent.LOUD):
        display.show(Image.HAPPY)
    elif microphone.current_event() == SoundEvent.QUIET:
        display.show(Image.ASLEEP)
    else:
        display.clear()
    sleep(500)
```
@microbit.withSensorUI


### Music

``` python
import music

music.play(['f', 'a', 'c', 'e'])
```
@microbit

### Pin logo

``` python
from microbit import *

while True:
    while pin_logo.is_touched():
        display.show(Image.HAPPY)
    display.clear()
```
@microbit


### Radio

``` python
from microbit import *
import radio


radio.on();
while True:
    message = radio.receive_bytes()
    if message:
        print(message)
```
@microbit.withSensorUI


### Random

``` python
from microbit import display, sleep
import random
import machine


n = random.randrange(0, 100)
print("Default seed ", n)
random.seed(1)
n = random.randrange(0, 100)
print("Fixed seed ", n)
print()
sleep(2000)
machine.reset()
```
@microbit


### Sensors

``` python
from microbit import *

while True:
    print(display.read_light_level())
    sleep(1000);
```
@microbit.withSensorUI

### Sensor effects (builtin)

``` python
from microbit import *

# You don't hear this one because we don't wait then play another which stops this one.
display.show(Image.HAPPY)
audio.play(Sound.GIGGLE, wait=False)
audio.play(Sound.GIGGLE)
display.show(Image.SAD) # This doesn't happen until the giggling is over.
audio.play(Sound.SAD)
```
@microbit

### Sensor effects (custom)

``` python
from microbit import *

# Create a Sound Effect and immediately play it
audio.play(
    audio.SoundEffect(
        freq_start=400,
        freq_end=2000,
        duration=500,
        vol_start=100,
        vol_end=255,
        waveform=audio.SoundEffect.WAVEFORM_TRIANGLE,
        fx=audio.SoundEffect.FX_VIBRATO,
        shape=audio.SoundEffect.SHAPE_LOG,
    )
)
```
@microbit

### Speech

``` python
import speech
import time

t = time.ticks_ms()
speech.pronounce(' /HEHLOW OHP AHL')
print(time.ticks_diff(time.ticks_ms(), t))
```
@microbit

### Stacksize

``` python
def g():
    global depth
    depth += 1
    g()
depth = 0
try:
    g()
except RuntimeError:
    pass
print('maximum recursion depth g():', depth)
```
@microbit

### Volume

``` python
from microbit import *
import music


music.pitch(440, wait=False)
while True:
  # Conveniently both 0..255.
  set_volume(display.read_light_level())
```
@microbit.withSensorUI


## Implementation

```` javascript
link:  style.css

@onload
window.createSensorUI = function (simulator, selector, state) {
  const parent = document.querySelector(selector)
  parent.classList.add('sensor-ui')
  parent.innerHTML = ''

  function onChange(id, value) {
    simulator.postMessage({ kind: 'set_value', id, value }, '*')
  }

  function makeCard(id) {
    const card = document.createElement('div')
    card.className = 'sensor-card'
    const label = document.createElement('span')
    label.innerText = id
    card.appendChild(label)
    return card
  }

  for (const sensor of Object.values(state)) {
    const { type, id } = sensor
    const card = makeCard(id)

    if (type === 'range') {
      const input = document.createElement('input')
      input.type = 'range'
      input.min = sensor.min
      input.max = sensor.max
      input.value = sensor.value
      input.addEventListener('input', (e) => onChange(id, e.target.value))
      card.appendChild(input)
    } else if (type === 'enum') {
      const select = document.createElement('select')
      for (const choice of sensor.choices) {
        const opt = new Option(
          choice,
          choice,
          choice === sensor.value,
          choice === sensor.value
        )
        select.appendChild(opt)
      }
      select.addEventListener('change', (e) => onChange(id, e.target.value))
      card.appendChild(select)
    } else {
      const info = { ...sensor }
      delete info.type
      const infoCode = document.createElement('code')
      infoCode.innerText = JSON.stringify(info)
      card.appendChild(infoCode)
    }

    parent.appendChild(card)
  }
}

@end


@microbit: @microbit_(@uid,```@input```,false)

@microbit.withSensorUI: @microbit_(@uid,```@input```,true)

@microbit_
<script>
const iframe = document.querySelector('#simulator_@0')
const simulator = iframe.contentWindow
let state = {}

const onMessage = (e) => {
    if (e.source !== simulator) return;
    const { data } = e;
    switch (data.kind) {
      case 'serial_output':
        console.stream(data.data);
        break;
      case 'log_output':
        if (data.headings) console.log(data.headings);
        if (data.data)     console.log(data.data);
        break;
      case 'radio_output':
        const text = new TextDecoder()
          .decode(e.data.data)
          .replace(/^\x01\x00\x01/, '')
        console.log(text)
        break
      case 'ready':
        state = data.state
        window.console.log('ready', data.state);
        if(@2) {
          createSensorUI(simulator, "#sensors_@0", state)
        }
        break
      case 'state_change':
        state = {
          ...state,
          ...data.change,
        }
        if(@2) {
          createSensorUI(simulator, "#sensors_@0", state)
        }
        break
      case 'log_delete':
        console.debug('[log_delete]');
        break;
      case 'request_flash':
        simulator.postMessage({
          kind: 'flash',
          filesystem: {
            'main.py': new TextEncoder().encode(`@1`),
          },
        }, '*');
        simulator.postMessage({
          kind: 'reset',
        }, '*');
        send.lia("LIA: terminal");
        break;
      case 'internal_error':
        console.error('Simulator internal error:', data.error);
        break;
      default:
        console.warn('Unknown message kind:', data.kind);
        break;
    }
  };

window.addEventListener('message', onMessage);

send.handle("input", (data) => {
  simulator.postMessage({
      kind: 'serial_input',
      data: data + "\r\n",
    },
    '*'
  )
})

send.handle("stop", () => {
    // detach the message listener
    window.removeEventListener('message', onMessage);

    // reset the iframe by reloading its src
    // (you can also set src="" then back, or completely remove/add the element)
    iframe.src = iframe.src;

    // optionally hide it if you want:
    iframe.style.display = "none";

    document.getElementById("sensors_@0").innerHTML = "";
  });

iframe.style.display = "block"

"LIA: wait"
</script>

<iframe
  id="simulator_@0"
  src="https://python-simulator.usermbit.org/v/0.1/simulator.html?color=green"
  title="Simulator"
  frameborder="0"
  scrolling="no"
  sandbox="allow-scripts allow-same-origin"
  style="width: 100%; max-width: 500px; aspect-ratio: 10 / 8; display: none"
></iframe>

<div id="sensors_@0" class="sensor-ui"></div>
@end
````
