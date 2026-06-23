# OpenJibo_Pulse
Jibo's lifeline. OpenJibo Pulse is a physical modding tool that simplifies server onboarding by lowering the barrier of entry to the Jibo Revival Group's AutoMod script. Just plug it in and monitor the matrix display.

### Bill of Materials (BOM)
#### Core Components
| Part Name | Manufacturer Part Number | Quantity | Unit Cost (USD) |
| :--- | :--- | :--- | :--- |
| Raspberry Pi Zero 2 W (SBC 1.0GHz 4-Core 512MB RAM) | SC1176 | 1 | *$17.99* |
| Pimoroni Unicorn HAT Mini (Expansion Board) | PIM498 | 1 | *$27.99* |

#### Hardware & DIY Cable Parts
| Part Name | Manufacturer Part Number | Quantity | Unit Cost (USD) |
| :--- | :--- | :--- | :--- |
| Adafruit Switchable USB Type A to Type A/MicroB/USB-C | 5972 | 1 | *$5.95* |
| DIY USB Cable Parts - Straight Type | 4108 | 1 | *$4.95* |
| DIY USB Cable Parts - Right Angle | 4105 | 1 | *$4.95* |
| Cable Jumper 3.94" | 3560 | 1 | *$1.50* |
| Oval Tactile Switch Cap (Black) | BTN K03 90 | 2 | *$0.59* |
| Oval Tactile Switch Cap (Green) | BTN K03 50 | 1 | *$0.82* |
| Oval Tactile Switch Cap (Red) | BTN K03 40 | 1 | *$0.54* |

### Pimoroni Unicorn HAT Mini (Manual Setup)
#### Debian Trixie (Latest Release)
### Step 1: Install the Core Library
```python
sudo pip3 install unicornhatmini --break-system-packages
```
### Step 2: Clone the Example Scripts
```python
cd ~
git clone https://github.com/pimoroni/unicornhatmini-python
```
### Step 3: Patch the Library Files
To stop the kernel validation crashes, modify the core library file to hand control over to the system SPI driver.
```python
sudo nano /usr/local/lib/python3.13/dist-packages/unicornhatmini/__init__.py
```
Apply the following three modifications:
#### A. Re-enable Hardware Chip Select
Locate the matrix initialization loop. Change `device.no_cs = True` to **`device.no_cs = False`**:
```python
for device, pin, offset in self.left_matrix, self.right_matrix:
    device.no_cs = False  # Changed from True
    device.max_speed_hz = spi_max_speed_hz
```
#### B. Bypass Strict Pin Setup Checks
Directly below that block, wrap the `GPIO.setup` call inside a `try...except` block so it doesn't crash on busy hardware lines:
```python
    try:
        GPIO.setup(pin, GPIO.OUT, initial=GPIO.HIGH)
    except Exception:
        pass
```
#### C. Disable Manual Software Pin Toggling
Use `Ctrl+W` to search for `def xfer`. Comment out (`#`) the manual software pin overrides. The underlying SPI hardware bus will now manage these line switches automatically:
```python
def xfer(self, device, pin, command):
    # Comment out manual software bit-banging:
    # GPIO.output(pin, GPIO.LOW)
    device.xfer2(command)
    # GPIO.output(pin, GPIO.HIGH)
```
Save and exit the editor (`Ctrl+O`, `Enter`, `Ctrl+X`).

Navigate to the `unicornhatmini-python` to test some examples! 🥳

### Much more coming soon... 👀
