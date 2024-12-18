# BLE-accelerator
import asyncio
from bleak import BleakScanner
import struct

# Threshold to determine motion vs stationary
ACCEL_THRESHOLD = 0.05  # Adjust based on real-world testing

# Function to process accelerometer data
def process_accelerometer_data(data):
    try:
        # Extract the accelerometer bytes (adjust index based on frame definition)
        x_bytes = data[8:10]
        y_bytes = data[10:12]
        z_bytes = data[12:14]

        # Convert bytes to signed integers (assuming little-endian, adjust if needed)
        x = struct.unpack('<h', x_bytes)[0] / 1000.0
        y = struct.unpack('<h', y_bytes)[0] / 1000.0
        z = struct.unpack('<h', z_bytes)[0] / 1000.0

        # Calculate magnitude of acceleration
        magnitude = (x**2 + y**2 + z**2)**0.5

        # Determine if the tag is stationary or moving
        if magnitude < ACCEL_THRESHOLD:
            return "Stationary"
        else:
            return "Moving"
    except Exception as e:
        print(f"Error processing data: {e}")
        return "Error"

# Callback function to handle BLE advertisements
def detection_callback(device, advertisement_data):
    # Check for specific manufacturer data or UUID
    manufacturer_data = advertisement_data.manufacturer_data
    for key, value in manufacturer_data.items():
        if key == 0xE1FF:  # Replace with actual manufacturer ID if needed
            state = process_accelerometer_data(value)
            print(f"Device: {device.address}, State: {state}")

# Main function
async def main():
    print("Starting BLE scanner...")
    scanner = BleakScanner()

    # Set callback for detection
    scanner.register_detection_callback(detection_callback)

    # Start scanning
    await scanner.start()

    try:
        await asyncio.sleep(30)  # Adjust scanning duration as needed
    finally:
        await scanner.stop()
        print("Scanning stopped.")

if __name__ == "__main__":
    asyncio.run(main())
