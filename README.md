
# Install the required dependencies using:
pip install pycrypto pyserial --user

Note: If errors concerning directories pop up when running of the command,
it would be best to use the full path directories rather than the relative paths

# Build the binary
```
make -f tensorflow/lite/micro/tools/make/Makefile \
TARGET=sparkfun_edge hello_world_bin 
```

To check if the binary build was successful run this code;

```
test -f tensorflow/lite/micro/tools/make/gen/ \
sparkfun_edge_cortex-m4/bin/hello_world.bin \
&& echo "Binary was successfully created" || echo "Binary is missing"
```
If you see "Binary was successfully created" printed to the console then you are good to go

# Sign the binary

First copy the file to a new one:
```
cp tensorflow/lite/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/ \
tools/apollo3_scripts/keys_info0.py \
tensorflow/lite/micro/tools/make/downloads/AmbiqSuite-Rel2.0.0/ \
tools/apollo3_scripts/keys_info.py
```
then create a signed binary, using this command:
```
python3 tensorflow/lite/micro/tools/make/downloads/ \
AmbiqSuite-Rel2.0.0/tools/apollo3_scripts/create_cust_image_blob.py \
--bin tensorflow/lite/micro/tools/make/gen/ \
sparkfun_edge_cortex-m4/bin/hello_world.bin \
--load-address 0xC000 \
--magic-num 0xCB -o main_nonsecure_ota \
--version 0x0
```
Now run this command tocreate a final version of the file:
```
python3 tensorflow/lite/micro/tools/make/downloads/ \
AmbiqSuite-
Rel2.0.0/tools/apollo3_scripts/create_cust_wireupdate_blob.py \
--load-address 0x20000 \
--bin main_nonsecure_ota.bin \
-i 6 \-o main_nonsecure_wire \
--options 0x1
```
# Flash the binary
Before attaching the device via USB, run the following command:
# macOS:
```
ls /dev/cu*
```
# Linux:
```
ls /dev/tty*
```

This should output a list of attached devices.

Now, connect the programmer to your computer’s USB port and run the previous
command again

You should see an extra item in the output, which is the name of your device(the board)
Take note of the name, it should look something like this "/dev/cu.wchusbserial-1450".

After you’ve identified the device name, put it in a shell variable for later
use:
```
export DEVICENAME=<your device name here>.
```

# Run the script to flash your board
First create an environment variable to specify the baud rate:
export BAUD_RATE=921600

Run this command:
```
python3 tensorflow/lite/micro/tools/make/downloads/ \
AmbiqSuite-Rel2.0.0/tools/apollo3_scripts/ \
uart_wired_update.py -b ${BAUD_RATE} \
${DEVICENAME} -r 1 -f main_nonsecure_wire.bin -i 6
```
Next, you’ll reset the board into its bootloader state and flash the board. 
On the board, locate the buttons marked RST and 14

Perform the following steps:
1. Ensure that your board is connected to the programmer and that the
entire thing is connected to your computer via USB.
2. On the board, press and hold the button marked 14 . Continue
holding it.
3. While still holding the button marked 14 , press the button marked
RST to reset the board.
4. Press Enter on your computer to run the script. Continue holding
button 14 .

You should now see some completed tasks appearing on your screen:

Keep holding button 14 until you see Sending Data Packet of length
8180 . You can release the button after seeing this (but it’s okay if you keep
holding it).

The program will continue to print lines on the terminal, until you see "Done",
which indicates a successful flashing.
