# Arduino-NFT-PUF

A joint project for the **IoT Security** and **Data Security** courses of the Master's Degree in Computer Science at the University of Salerno.

## Overview

**Arduino-NFT-PUF** is an innovative approach to device fingerprinting and ownership verification by leveraging **SRAM Physical Unclonable Functions (PUFs)** and **blockchain technology**. This project enables unique identification of Arduino devices through their intrinsic hardware characteristics and secures ownership via **Non-Fungible Tokens (NFTs)**.

This project is a **proof of concept** and is not intended for production use. It demonstrates how hardware-based security can be combined with blockchain technology to create a secure and verifiable ownership system.

## Key Features

The project consists of three main components:

### 1. Arduino SRAM-PUF
- **Physical Unclonable Function (PUF)** implementation using the inherent randomness in SRAM startup values.
- A custom bootloader and data manipulation are required to obtain a stable PUF response.

### 2. NFT Smart Contract
- A simple **NFT Smart Contract** compliant with the **ERC-721 standard**.
- Links an Ethereum account to a PUF for ownership verification.
- Written in **Solidity** and tested on an Ethereum testnet using **Hardhat**.

### 3. DApp (Decentralized Application)
- A simple DApp to interact with the Smart Contract.
- Users can log in via **MetaMask** and mint NFTs through the app.
- Built using **Web3.js** for blockchain interaction.

## Prerequisites

### Hardware Requirements
- Two **Arduino Uno R3** boards.
- 6 cables to connect the boards.
- A USB cable to connect the board to a PC.

### Software Requirements
#### For the Arduino SRAM PUF:
- **Arduino IDE**
- **avrdude**
- **avr-gcc**
- **scons**
- **minicom**

#### For the NFT Smart Contract:
- **Hardhat** (for testing on a testnet)
- **Solidity**

#### For the DApp:
- **JavaScript**
- **Web3.js**

## Important Notes

- **Arduino as ISP**: Works on **Arduino Uno R3** boards. Tested on **Arduino Uno R4** without success.
- **Operating Systems**: Tested on **Ubuntu** and **Manjaro**. Issues were encountered on **MacBooks with Apple Silicon**. Windows has not been tested. We recommend using **Ubuntu Linux** for the PUF steps.
- **Proof of Concept**: This project is a **proof of concept**. We do not recommend deploying it on a production network or the mainnet. No liability is assumed for any use of this project.

## How to Run

### 1. Generating an SRAM-PUF

A **Physical Unclonable Function (PUF)** leverages the intrinsic randomness of the hardware to derive a unique identifier. To generate the PUF, we use the randomness of the SRAM at startup. This requires overwriting the original bootloader, as it initializes the SRAM, preventing access to the startup SRAM data.

#### Using an Arduino Board as an ISP
To load a custom bootloader, one Arduino board will act as an **ISP (In-System Programmer)** to program another board (the "target" board).

1. **Set up the Programmer Board**:
   - Connect one Arduino board to the PC.
   - Open the **Arduino IDE**.
   - Go to **File -> Examples -> 11. ArduinoISP** and load the `ArduinoISP` sketch onto the board.
   - Go to **Tools -> Programmer** and select **Arduino as ISP**.

2. **Connect the Programmer and Target Boards**:
   - Use the following pin connections:

     | **Programmer Pins** | **Target Pins** |
     |---------------------|-----------------|
     | PIN 10              | RESET           |
     | PIN 11              | PIN 11          |
     | PIN 12              | PIN 12          |
     | PIN 13              | PIN 13          |
     | 5V                  | 5V              |
     | GND                 | GND             |

#### Compiling and Loading the Custom Bootloader
1. Download the content of the `Arduino PUF` folder.
2. Connect the Programmer board to the Target board using the above table.
3. Connect the Programmer board to the PC.
4. Navigate to the `scripts` folder and compile the bootloader using:
   ```bash
   ./compile_bootloader.sh
   ```
   The compiled bootloader will be available in the `build` folder.
5. Flash the bootloader using:
   ```bash
   ./flash_bootloader.sh <path to bootloader.hex>
   ```
   Make sure to use the correct file.

#### Obtaining Two Unstable PUFs
1. Disconnect the Target board from the Programmer board.
2. Disconnect the Programmer board from the PC.
3. Connect only the Target board to the PC.
4. Check the serial port using:
   ```bash
   ls /dev/tty:.*
   ```
   Note the port name (e.g., `/dev/ttyUSB0` or `/dev/ttyACM0`).
5. Open two terminals and run `minicom` on both:
   On the first terminal:
   ```bash
   minicom -D /dev/tty.usb_portname -b 115200 -C file1.txt
   ```
   On the second terminal:
   ```bash
   minicom -D /dev/tty.usb_portname -b 115200 -C file2.txt
   ```
   Make sure to disconnect and reconnect the Target board between `minicom` executions.
7. Extract the unstable PUFs from the two files and save them as `enrollement/dump/new_arduino_1_0.puf` and `enrollement/dump/new_arduino_1_1.puf`.

#### Obtaining a Stable PUF
1. Run the enroller script:
   ```bash
   ./enroller > helperdata.txt
   ```
2. Extract the `helper_data` field from `helperdata.txt`. Convert it by adding `0x` before each byte and separating them with commas (e.g., `466f727a61204e61706f6c69` becomes `0x46, 0x6f, 0x72, 0x7a, 0x61, 0x20, 0x4e, 0x61, 0x70, 0x6f, 0x6c, 0x69`).
3. Open the `bootloader/puf.c` file and insert the converted helper data:
   ```c
   const uint8_t helper_data[304] PROGMEM = {/*insert here the converted helper data*/};
   ```
4. Compile the modified bootloader and burn it on the Target board.
5. Extract the stable PUF:
   - Connect the Target board to the PC.
   - Run `minicom`:
     ```bash
     minicom -D /dev/tty.usb_portname -b 115200 -C puf_response.txt
     ```
   - Type `x` in the `minicom` terminal to print the stable PUF.
   - The stable PUF will be saved in `puf_response.txt`.
     
Now the stable PUF can be used as desired.

### 2. Deploying the NFT on Hardhat Testnet

1. **Initialize a Hardhat Project**:
   - Create a new directory and initialize an npm project:
     ```bash
     npm init
     ```
   - Install Hardhat:
     ```bash
     npm install --save-dev hardhat
     ```
   - Install dependencies:
     ```bash
     npm install @openzeppelin/contracts
     ```
   - Initialize a Hardhat project:
     ```bash
     npx hardhat init
     ```
     Choose the **JavaScript** project.

2. **Set Up the Smart Contract**:
   - Delete the sample contract and deployment scripts.
   - Download the Smart Contract and place it in the `contracts` folder.
   - Download the deployment script and place it in `ignition/modules/`.

3. **Compile and Deploy**:
   - Compile the contract:
     ```bash
     npx hardhat compile
     ```
   - Run the Hardhat testnet:
     ```bash
     npx hardhat node
     ```
   - Deploy the contract:
     ```bash
     npx hardhat ignition deploy ignition/modules/ArduinoNFT.js --network localhost
     ```
The contract is now deployed, and you can interact with it.

### 3. Interacting with the NFT
Make sure MetaMask is installed, the Hardhat node is running and the Smart Contract is deployed.
1. **Set up the DApp**:
    - Install Web3.js:
        ```bash
        npm install web3
        ```
    - Make sure to insert the Smart Contract address into the const `contractAddress` variable of the `index.html` file.

2. **Run the DApp**:
    - To run the DApp, go to the root of the project and start a webserver:
      ```bash
      npx watch-http-server DApp/
      ```
Now you can interact with the DApp to mint NFTs.

## Authors

This project was developed by:
- **Simone D'Angelo**
- **Mariano Aponte**
- **Sergio Aprea**

## Acknowledgments

We extend our gratitude to **Dr. Franco Cirillo** for providing the hardware and software needed to implement the Arduino SRAM-PUF.
