# Networked Embedded Systems

## 3: Node Architecture

RSSI: received signal strenght indicator

Noise figure (db), NF = SNRI/SNRO, ratio between input and output of the transceiver electronic circuit

DSSS: direct sequence spread spectrum (?)

Amplifier power: P_amp = α_amp + β_amp*P_tx.
P_tx is the radiated power.
Amplifier efficiency: η = P_tx/P_amp

Energy for transmission of n bits: E_tx(n) = T_start * P_start + n/(R*R_code) * (P_txElec + P_amp)

* T_start/P_start: time/power to leave sleep mode
* R: nominal data rate
* R_code: coding rate
* P_txElec: transmitter electronics power

#### Example

Consumption: 11.3mA at P_tx=0dBm

P[dBm] = 10*log(P_mW)

P_tx = 0dBm = 1mW

P_txElec + P_amp = 11.3mA * 3.3V = 37.3mW

Packet length = 32 bytes
Data rate (R) = 2 Mbps
R_code = 1
T_start = 130μs
P_start = 7mA * 3.3V = 23mW

E_tx = 130e-6 * 23 + (32 * 8) / (2e6 * 1) * 37.3 = 7.76e-3 mJ



## 4: Physical Layer


### Modulation

* FSK: frequency shift keying
* ASK: amplitude shift keying
* PSK: phase shift keying
  * BPSK: binary phase shift keying, shifts of 0 or π
  * QPSK: four phase shifts (0, π/2, π, 3π/2), 4-ary modulation, one symbol represents 2 bits of data


### Signal distortion

* Attenuation: uitdijing
* Reflection
* Diffraction: start new wave from a sharp edge
* Scattering: rough surfaces
* Doppler fading: shift in frequencies due to movement


#### Attenuation

P_recv(d) = P_recv(d_0) * (d_0/d)^γ

γ is the path-loss exponent, between 2 and 6, for other environments than free space

In logarithmic form: PL(d)[dB] = PL

### From waves to bits

* SNR: signal to noise ratio
* E_b / N_0 = SNR * (B / R), B=channel bandwidth, R=channel bit rate, dimensionless but usually expressed in dB
* BER: bit error rate
* PER: p_p = 1 - (1-p_b)^N, p_p=packet error probability, p_b=BER, N=packet length in bits

E_b/N_0 (dB) = P_signal(dB) - P_noise(dB) + 10log(B/R)


## 5: Link Quality Estimation

### In hardware

RSSI: received signal strenght indicator, mW or dBm

SNR = P_signal/P_noise

SNR_dB = 10log(SNR) = P_signal,dB - P_noise,dB

LQI: link quality indicator

Advantages: no extra computation, fast, inexpensive

Limitations: only able to identify as good or bad, only measured for successfully received signals, not available on all radio transceivers.

### In software

PRR: packet reception ratio, using a moving average, ω

TWMA: time weighted moving average

EWMA: exponentially weighted moving average, α: coefficient for decreasing weights, EWMA^τ = α * Link_τ + (1-α) * EWMA^(τ-1)

WMEWMA: window mean exponentially moving average, PRR but smoothed using EWMA with a coefficient, so WMEWMA=α * WMEWMA_prev + (1-α) * PRR

ETX: expected transmission count, taking into account link asymmetry, ETX=1/(PRR_up * PRR_down). Parameter ω: window size for PRR estimation.

ETX of a route: sum of ETX for each link


## 6: Medium Access Control

Energy wastage problems:

* Collisions
* Overhearing: not meant for you
* Idle listening
* Protocol overhead: packet header+footer

Contention-free: ensuring exclusive access

* FDMA: frequency division multiple access
* TDMA: time division
* CDMA: code division

Contention-based: simpler

* ALOHA: uses acknowledgements, e.g. exponential back-off in case of collisions.
* Slotted-ALOHA: nodes may only commence transmission at predefined points in time, more efficient, synchronization needed
* CSMA: carrier sense
  * p-persistent CSMA: node continuously listens until idle, then sends with probability p
  * With back-off: if idle, waits randomly in contention window, if transmission error, increases contention window with binary exponential back-off.


CSMA problems:

* Hidden-terminal: A and C can reach B but not each other
* Exposed-terminal: A reaches B but also C and D which are transmitting between each other, unnecessary waiting

MACA: multiple access collision avoidance, request-to-send (RTS) and clear-to-send (CTS) packets

Contention-based:

* Sensor-MAC (S-MAC): duty cycling, schedule synchronization using SYNC packets, overhearing avoidance using RTS/CTS, message passing where a long packet is split and each fragment is acknowledged.
* Timeout-MAC (T-MAC): S-MAC variation, adapting to traffic load
  * TA: active period, must be long enough to receive at least the start of a CTS packet (it may not hear the RTS)
  * Early sleeping problem for 4 distinct nodes, solution: future-request-to-send (FRTS)
* LPL: low power listening, sender uses a long preamble, receivers periodically sample with a fixed period
* WiseMAC: for downlink, from base to sensor nodes, ALOHA based, preamble sampling
* Berkeley-MAC (B-MAC): CSMA, no RTS/CTS, LPL, uses more reliable Clear Channel Assessment (CCA) which dynamically adjusts the threshold
* XMAC: improvements on B-MAC, target address in preample to minimize overhearing, strobed preamble with acknowledgment in between

Contention-free:

* Traffic-Adaptive Medium Access (TRAMA): adaptive election algorithm (AEA) using global hash function, a node calculates priority for itself and neighbors to see if it has highest priority.
* Lightweight MAC (LMAC): slots divided over 2-hop neighborhood using a slot occupation bit-mask, not suitable for mobile networks as collision detection and recovery is time consuming.
* Low-Energy Adaptive Clustering Hierarchy (LEACH): needs cluster heads that communicate with base station, higher energy requirements, cluster head is rotated, cluster heads broadcast advertisements, nodes can join, in steady-state phase the cluster head remains awake to receive sensor data and send to base station.

Hybrid:

* Zebra MAC (Z-MAC): send explicit contention notification (ECN) message when high contention is detected, then it switched to high contention level (HCL) mode with TDMA, if no ECN received after a certain amount of time it switched back to low contention level (LCL) with CSMA.





## 8: IEEE 802.15.4

* OQPSK, DSSS
* 62.5 ksymbol/sec, 4 bits per symbol, data rate 250 kbps
* Superframe duration = BSD (base slot duration) * 2^SO
* Beacon interval = BSD * 2^BO
* Contention Access Period (CAP) and Contention Free Period (CFP) with Guaranteed Time Slots (GTS)
* TSCH: Time Slotted Channel Hopping
  * ASN: absolute slot number

## 9: IoT protocols

Short range:

* ZigBee
* 6TiSCH
* BLE
* WiFi

Long range:

* Sigfox
* LoRa

IEEE 802.15.4: 16 channels, 250 kbit/s, channel width: 2 MHz, channel separation: 5 MHz, 

