
# MeshAlyzer Device
<div align="center">
  <img src="https://raw.githubusercontent.com/The-Lake-Lab/MeshAlyzer/refs/heads/main/img/teampicmesh.jpg" alt="MeshAlyzer Team Picture" width="450"/>
  <br>
  <img src="https://raw.githubusercontent.com/The-Lake-Lab/MeshAlyzer/refs/heads/main/img/UIoverview.png" alt="MeshAlyzer UI Overview" width="450"/>
</div>

## Table of Contents

* [Introduction](#introduction)  
* [Installation](#installation)  
* [Usage](#usage)  
* [Contributing](#contributing)  
* [License](#license)  
* [Contact](#contact)  
* [Acknowledgements](#acknowledgements)  

---

## Introduction

The **MeshAlyzer** is an advanced testing device designed to simulate realistic coughing forces and pressures on a porcine abdominal wall. It enables researchers and surgeons to evaluate hernia mesh materials under dynamic conditions, facilitating informed decisions for hernia repair procedures.  

### Key Features
- **Pressure Control System**: Integrated pressure sensors with ADCs for precise measurements. 
- **Valve System**: Two 3-position, 5-way valves controlled by four relays.  
- **User Interface**: Customizable testing protocols and real-time data visualization.  
- **Data Logging**: Continuous recording of test data for in-depth analysis.  
- **Tissue Simulation**: Mimics coughing pressures with interchangeable balloon materials.  

---

## Installation
The MeshAlyzer device requires a Raspberry Pi 5 or Raspberry Pi 3 for operation. Follow the instructions below to set up your device.

### Raspberry Pi 5 Setup
1. Clone the repository:
   ```bash
   git clone https://github.com/The-Lake-Lab/MeshAlyzer.git
   cd MeshAlyzer
   ```

2. Install dependencies:
   ```bash
   sudo apt-get install python3-lgpio
   ```
3. Set up the virtual environment (optional but recommended):
   ```bash
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
   ```
4. Connect hardware components:
   - Link pressure sensors to ADCs as shown in the wiring diagram.
   - Ensure relays and valves are connected to the Raspberry Pi GPIO pins as specified.

5. Launch the application (from the `MeshAlyzer` folder, with the venv active):
   ```bash
   cd MeshAlyzer
   source venv/bin/activate
   python main.py
   ```
### Raspberry Pi 3 setup
The Pi 3 reads the pressure transducers and streams them to the Pi 5 over TCP
(port 65432) using `PressureSensorReader_rasp3.py`. Run it automatically at boot as a systemd service.

1. Create the service file:
```bash
sudo nano /etc/systemd/system/spidev-load.service
```
add:
```ini
[Unit]
Description=Pressure Sensor Client
After=network.target

[Service]
ExecStartPre=/sbin/modprobe spidev
ExecStart=/usr/bin/python3 /home/lakelab/MeshAlyzer/PressureSensorReader_rasp3.py
Restart=always
User=lakelab
WorkingDirectory=/home/lakelab
StandardOutput=inherit
StandardError=inherit

[Install]
WantedBy=multi-user.target
```
2. Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable spidev-load.service
sudo systemctl start spidev-load.service
```

### ADC Setup
The MeshAlyzer uses a Waveshare ADS1263 High-Precision AD HAT for pressure sensing
(driver: `ADS1263.py`, pins: `config.py`).

1. Wiring (ADS1263 → Raspberry Pi 3), as set in `config.py`:
   - DIN (MOSI) → GPIO 10 (SPI0 MOSI)
   - DOUT (MISO) → GPIO 9 (SPI0 MISO)
   - SCLK → GPIO 11 (SPI0 SCLK)
   - CS (Chip Select) → GPIO 22
   - DRDY (Data Ready) → GPIO 17
   - RESET → GPIO 18

2. Enable SPI (`sudo raspi-config` → Interface Options → SPI) and install the libraries:
```bash
sudo apt-get install python3-spidev python3-rpi.gpio
```


## Usage

1. **Setup**:
   - Place the porcine tissue sample on the testing platform.
   - Connect the device to a power source and ensure all components are operational.

2. **Run Tests**:
   - Open the user interface and select a protocol.
   - Start the simulation and observe the test in progress.

3. **Analyze Data**:
   - Access the recorded data logs for performance evaluation.
   - Use the insights to determine optimal mesh materials.

---

## Contributing

We welcome contributions to enhance the MeshAlyzer project!  

### How to Contribute
1. Fork the repository.
2. Create a new branch for your feature or bugfix:
   ```bash
   git checkout -b feature-name
   ```
3. Commit your changes:
   ```bash
   git commit -m "Description of your changes"
   ```
4. Push your branch and create a pull request:
   ```bash
   git push origin feature-name
   ```

---

## License

This project is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.

---

## Contact

For questions or feedback, please contact:  
- **Cole Hanan**  
- **Alexander Gadin**  
- **Evan Maples**  

---

## Acknowledgments

We would like to thank:  
- Dr. Spencer Lake for his guidance and support.  
- The BME Department at Washington University in St. Louis for resources and assistance.  
- The Chemistry Machine Shop for their technical expertise.  
