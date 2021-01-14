# Cyberattacks, Crime and Defences (2020 Q2)

## Lectures

### 1b

* Know OWASP Top Ten, know meaning of first 5 and XSS.
* Watering Hole attack
* Phishing/spear-phishing

### 2b

* Know what an exploit is. Know what a zero-day exploit is.

### 4b

* Equifax Hack: need to know.

### 6a

* Need to know: meterpreter.
* Supply-chain attack: attack suppliers of target companies.


### 7b

* Cyber resilience vs. cyber security: resilience is that the business can keep running while under attack, cyber security is that we are defended against attacks.
* Mirai: DDoS botnet with IoT devices.
* Will get worse: due to move to IP-based IoT.
* How detect attack: rejection based, blacklist known attacks vs acceptance-based: determine good behaviour, mark the rest as attacks.
* Whitelisting systems: maintaining them is a pain in the neck because the software/good behaviour always changes.

### 8

* Anomaly detection only works if systems behave 'predictably'.
* IT/OT in ICS networks: IT very much standardised (HTTP/SSH/..), OT not so much
* Blacklisting viruses for ICS is not feasible: too many different systems (Siemens/Honeywell/..), not commercially viable to create blacklists for these systems which are only used in small number of places.

* Intrusion prevention: not feasible
* Intrusion detection:
  * Knowledge-based: not feasible because of adaptability.
  * Behaviour-based:
    * Specification-based: largely insufficient but promising in specific small areas.
    * Anomaly-based systems:
      * BlackBox: machine learning, semantics do not correlate, i.e. alerts will not be meaningful, explainability is a problem. Largely insufficient.
      * WhiteBox: it works, but on specific systems. It's not quite about intrusion detection, more about being able to understand what happens in the target system. In normal IT systems, the systems are too complex to be understandable.

Conclusion: specification-based and WhiteBox are the only way for prevention.

"I believe that today the single most important reason why attacks are so difficult to counter is that present systems are so hard to monitor.

"I believe the only practical way towards making more secure systems goes through making software more supervisable."


### 9

* Encryption can work against security: anomalies on the network are less visible.
* Encryption is not often needed, it's only necessary for private data.

### 10

* Attacks don't care about high vulnerabilities: top 10 CVEs are even likely used as 10 random CVEs.
* Weakest link model of attacks doesn't hold up. It ignores scale (attackers-victims are not one-to-one).
* Types of attackers: criminals (cost < benefits), hacktivist (cost < fixed limit), nation states (no constraints), occasionals (insiders).

### 12

Cybercrime is an extension of traditional crime.

* First order: enabler of the attack
* Second order: products/information, e.g. credit card numbers, banking account numbers, ..

Agency theory

Legal market has law enforcement and regulation that enforces fair market and competition. This is not available in the criminal malware market.

#### Forums

Moral hazard: people not sticking to the contract, easy because of anonimity.

Mechanisms to address moral hazards

* High cost of entry: being banned is expensive
* Fear of punishment (being kicked out of the market)
* User reputation
* Rule enforcements using trials

Forum allows for high interaction between seller and buyer.

#### E-commerce illegal markets

### 13

Cybercriminal networks:

* are international *and local* (offenders and victims are physically located)
* *have* dependency relationships
* *last long*

Members of cybercriminal networks:

* know each other based on *offline and online social ties*
* are *mostly not* specialists
