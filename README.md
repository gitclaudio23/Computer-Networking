# Computer-Networking

# Proyek Ping Python di Raspberry Pi

Proyek ini adalah implementasi dari program ping menggunakan Python yang dapat dijalankan pada Raspberry Pi dengan sistem operasi Linux. Program ini mengirimkan paket ICMP (ping) ke alamat IP tujuan, dan menghitung waktu round-trip dari paket yang dikirim.

## Persyaratan

- Raspberry Pi dengan sistem operasi berbasis Linux (misalnya Raspberry Pi OS).
- Python 3.x terinstal pada Raspberry Pi.
- Hak akses root atau kemampuan untuk menjalankan socket raw (untuk mengirim paket ICMP).
- Modul Python berikut:
  - `socket` (bawaan Python)
  - `os` (bawaan Python)
  - `sys` (bawaan Python)
  - `struct` (bawaan Python)
  - `time` (bawaan Python)
  - `select` (bawaan Python)

## Langkah-langkah Menjalankan Proyek

### 1. Persiapkan Raspberry Pi

Sebelum menjalankan script, pastikan Raspberry Pi Anda sudah terinstal Python 3. Anda dapat mengeceknya dengan perintah berikut:

```bash
python3 --version

2. Membuat dan Menyimpan Script
Buka terminal pada Raspberry Pi.

Buat file baru untuk menyimpan script Python dengan perintah berikut:

bash
Copy code
nano ping.py
Salin dan tempelkan kode berikut ke dalam file ping.py:

python
Copy code
from socket import *
import os
import sys
import struct
import time
import select

ICMP_ECHO_REQUEST = 8

def checksum(string):
    csum = 0
    countTo = (len(string) // 2) * 2
    count = 0

    while count < countTo:
        thisVal = string[count + 1] * 256 + string[count]
        csum += thisVal
        csum &= 0xffffffff
        count += 2

    if countTo < len(string):
        csum += string[len(string) - 1]
        csum &= 0xffffffff

    csum = (csum >> 16) + (csum & 0xffff)
    csum += (csum >> 16)
    answer = ~csum
    answer &= 0xffff
    answer = answer >> 8 | (answer << 8 & 0xff00)
    return answer

def receiveOnePing(mySocket, ID, timeout, destAddr):
    timeLeft = timeout
    while True:
        startedSelect = time.time()
        whatReady = select.select([mySocket], [], [], timeLeft)
        howLongInSelect = (time.time() - startedSelect)
        if whatReady[0] == []: # Timeout
            return None

        timeReceived = time.time()
        recPacket, addr = mySocket.recvfrom(1024)

        # Extract ICMP header from IP packet
        icmpHeader = recPacket[20:28]
        type, code, checksum, packetID, sequence = struct.unpack("bbHHh", icmpHeader)

        if packetID == ID:
            bytesInDouble = struct.calcsize("d")
            timeSent = struct.unpack("d", recPacket[28:28 + bytesInDouble])[0]
            return timeReceived - timeSent

        timeLeft -= howLongInSelect
        if timeLeft <= 0:
            return None

def sendOnePing(mySocket, destAddr, ID):
    myChecksum = 0

    # Create dummy header with checksum = 0
    header = struct.pack("bbHHh", ICMP_ECHO_REQUEST, 0, myChecksum, ID, 1)
    data = struct.pack("d", time.time())

    # Calculate checksum
    myChecksum = checksum(header + data)

    # Correct checksum for network byte order
    if sys.platform == 'darwin':
        myChecksum = htons(myChecksum) & 0xffff
    else:
        myChecksum = htons(myChecksum)

    # Final packet with correct checksum
    header = struct.pack("bbHHh", ICMP_ECHO_REQUEST, 0, myChecksum, ID, 1)
    packet = header + data

    mySocket.sendto(packet, (destAddr, 1))

def doOnePing(destAddr, timeout):
    icmp = getprotobyname("icmp")
    mySocket = socket(AF_INET, SOCK_RAW, icmp)

    myID = os.getpid() & 0xFFFF
    sendOnePing(mySocket, destAddr, myID)
    delay = receiveOnePing(mySocket, myID, timeout, destAddr)

    mySocket.close()
    return delay

def ping(host, timeout=1, count=4):
    dest = gethostbyname(host)
    print(f"Pinging {dest} using Python:")
    print("")

    delays = []

    for i in range(count):
        delay = doOnePing(dest, timeout)
        if delay is None:
            print("Request timed out.")
        else:
            delay = delay * 1000
            delays.append(delay)
            print(f"Reply from {dest}: time={delay:.2f} ms")
        time.sleep(1)

    # Display statistics
    packet_sent = count
    packet_received = len(delays)
    packet_loss = ((packet_sent - packet_received) / packet_sent) * 100

    print("\nPing statistics for", dest)
    print(f"    Packets: Sent = {packet_sent}, Received = {packet_received}, Lost = {packet_sent - packet_received} ({packet_loss:.0f}% loss)")

    if delays:
        print("Approximate round trip times in milli-seconds:")
        print(f"    Minimum = {min(delays):.2f} ms, Maximum = {max(delays):.2f} ms, Average = {sum(delays) / len(delays):.2f} ms")

# Change the host to 127.0.0.1
ping("127.0.0.1")
Simpan dan keluar dari editor dengan menekan CTRL + X, kemudian tekan Y dan Enter.

3. Memberikan Hak Akses Root (Jika Diperlukan)
Karena script ini menggunakan socket raw untuk mengirimkan paket ICMP, Anda memerlukan hak akses root. Gunakan perintah sudo untuk menjalankan script:

bash
Copy code
sudo python3 ping.py
4. Jalankan Script
Setelah memberi hak akses, Anda bisa menjalankan script ping menggunakan perintah:

bash
Copy code
sudo python3 ping.py
Script ini akan memulai ping ke alamat IP lokal (127.0.0.1) dan menampilkan statistik hasil ping.

5. Hasil yang Diharapkan
Setelah menjalankan perintah di atas, Anda akan melihat output seperti berikut:

sql
Copy code
Pinging 127.0.0.1 using Python:

Reply from 127.0.0.1: time=0.36 ms
Reply from 127.0.0.1: time=0.24 ms
Reply from 127.0.0.1: time=0.20 ms
Reply from 127.0.0.1: time=0.22 ms

Ping statistics for 127.0.0.1
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Approximate round trip times in milli-seconds:
    Minimum = 0.20 ms, Maximum = 0.36 ms, Average = 0.26 ms
Catatan
Jika Anda menerima pesan kesalahan terkait izin saat menjalankan script, pastikan Anda menjalankannya dengan sudo karena script ini memerlukan hak akses root untuk mengirimkan paket ICMP.
Anda dapat mengubah alamat IP yang diping dengan mengganti parameter pada baris ping("127.0.0.1") sesuai kebutuhan.
