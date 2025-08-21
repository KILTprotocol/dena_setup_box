# Connection and Setup

This document provides instructions on how to connect to the device, copy the necessary files, and set up the `ssi-server.service` systemd service.

## Prerequisites

- Access to [https://app.remote.it/#/](https://app.remote.it/#/)
- A `.ppk` file for each device you want to connect to
- PuTTY installed on your machine
- SSH client installed on your machine

## Connecting to the Device

1. Go to [https://app.remote.it/#/](https://app.remote.it/#/)
2. Select the device you want to connect to and press connect for SSH.

**Note:** Each time a connection is started, the host and port are changing.

### Using PuTTY

1. In the [Session] section, paste the right hostname and port (from the web page).
2. In [Connection/SSH/Auth/Credentials], paste the right `.ppk` file.
3. Press open. You may be asked for credentials.

### Using SSH 

1. Convert `.ppk` file into openSSH compatible file format:

    ```bash
    puttygen oli_box_thomas.ppk -O private-openssh -o oli_box_thomas_openssh
    ```

2. Check permission of the file:

    ```bash
    chmod 600 oli_box_thomas_openssh
    ```

3. Connect by:

    ```bash
    ssh -i oli_box_thomas_openssh user@hostname -p port
    ```

## Copying Files to the Device

1. Copy the `ssi-server` binary to the `/home/pi/ssi-server` directory.

    ```bash
    scp -P PORT -i SSH_IDENTITY_FILE dive-spiritnet pi@HOST:/home/pi/ssi-server 
    ```

2. Copy the needed frontend to the `/home/pi/dist` directory.

    ```bash
    scp -P PORT -i SSH_IDENTITY_FILE -r dist pi@HOST:/home/pi/dist 
    ```

## Setting Up the SystemD Service

1. Copy the `ssi-server.service` file to the `/etc/systemd/system` directory.

    ```bash
    scp -P PORT -i SSH_IDENTITY_FILE ssi-server.service pi@host:~/ssi-server.service
    ssh -p PORT -i SSH_IDENTIY_FILE pi@host 'sudo mv ~/ssi-server.service /etc/systemd/system/ssi-server.service'
    ```

2. Connect to the box and reload the systemd daemon to recognize your new service:

    ```bash
    sudo systemctl daemon-reload
    ```

3. Enable the service to start on boot:

    ```bash
    sudo systemctl enable ssi-server.service
    ```

4. Start the service:

    ```bash
    sudo systemctl start ssi-server.service
    ```

5. Check the status of the service:

    ```bash
    sudo systemctl status ssi-server.service
    ```

6. Optional: you can follow the logs by: 

    ```bash
    sudo journalctl -u ssi-server.service -f
    ```

## Testing 

Ensure that the binary is running and the frontend is exposed at port 3333. Verify all functionalities and reset the box if all tests are successful:
    - Create a Box DID
    - Create a Claim 
    - Go to the attester service and approve the claim for the box
    - Verify, if the claim is approved now
    - Create a self issued credential (Selbstauskunfszertifikat)
    - Create a DID for the user (Betreiber)
    - Participate to one use case (As long as EnergyWeb Foundation is not exposing their use case service, you can only verify if the service endpoint of the DID got updated.)

## Building the Binary 

To build the binary, navigate to the dive [repository](https://github.com/BTE-Trusted-Entity/dive) and execute the following command:

```bash
BUILD_ARGS="--features=spiritnet" make release-build   
```
