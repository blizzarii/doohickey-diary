from machine import Pin
import time
import network
import ntptime

# --- UPDATE YOUR WI-FI CREDENTIALS HERE ---
SSID = "BTHub3-JW5J"
PASSWORD = "cb3a9bc76f"

# Pin definitions matching your 12-wire setup
RS = Pin(19, Pin.OUT)
E  = Pin(23, Pin.OUT)
D4 = Pin(25, Pin.OUT)
D5 = Pin(26, Pin.OUT)
D6 = Pin(27, Pin.OUT)
D7 = Pin(33, Pin.OUT)

def pulse_enable():
    E.value(0)
    time.sleep_us(1)
    E.value(1)
    time.sleep_us(1)
    E.value(0)
    time.sleep_us(100)

def write_4bits(data):
    D4.value((data >> 0) & 1)
    D5.value((data >> 1) & 1)
    D6.value((data >> 2) & 1)
    D7.value((data >> 3) & 1)
    pulse_enable()

def send_instruction(data):
    RS.value(0)
    write_4bits(data >> 4)
    write_4bits(data & 0x0F)

def send_char(data):
    RS.value(1)
    write_4bits(data >> 4)
    write_4bits(data & 0x0F)

def lcd_init():
    time.sleep_ms(50)
    RS.value(0)
    E.value(0)
    
    for _ in range(3):
        write_4bits(0x03)
        time.sleep_ms(5)
    write_4bits(0x02)
    
    send_instruction(0x28) # 4-bit, 2 lines
    send_instruction(0x0C) # Display ON, cursor OFF
    send_instruction(0x06) # Increment cursor
    send_instruction(0x01) # Clear screen
    time.sleep_ms(2)

def lcd_print(text):
    for char in text:
        send_char(ord(char))

# Initialize LCD display
lcd_init()
send_instruction(0x80)
lcd_print("Connecting WiFi")

# Connect to Wi-Fi network
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect(SSID, PASSWORD)

dot_count = 0
while not wlan.isconnected():
    time.sleep(0.5)
    dot_count = (dot_count + 1) % 4
    send_instruction(0xC0)
    lcd_print("Loading" + "." * dot_count + "   ")

# Sync time via NTP servers
send_instruction(0x01)
send_instruction(0x80)
lcd_print("Syncing NTP Time")

try:
    ntptime.settime()
    time.sleep(1)
except Exception as e:
    print("NTP sync error:", e)

# Clear screen and prepare clock view
send_instruction(0x01)

# Timezone adjustment (Set to 0 for GMT/UTC, or adjust for your local offset in hours)
GMT_OFFSET = 0 

while True:
    # Calculate current local time from system ticks + NTP baseline
    t = time.localtime(time.time() + (GMT_OFFSET * 3600))
    year, month, mday, hour, minute, second, _, _ = t
    
    date_str = "{:02d}/{:02d}/{:04d}".format(mday, month, year)
    time_str = "{:02d}:{:02d}:{:02d}".format(hour, minute, second)
    
    # Row 0: Date
    send_instruction(0x80)
    lcd_print("Date: " + date_str)
    
    # Row 1: Time
    send_instruction(0xC0)
    lcd_print("Time: " + time_str)
    
    time.sleep(1)
