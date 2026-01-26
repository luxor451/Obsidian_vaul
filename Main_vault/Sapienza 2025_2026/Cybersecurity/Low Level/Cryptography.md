![[1.pdf]]
# Overview
Cryptography $\neq$ Security
It deals with the secrecy of information
Encryption does not live alone without authentification

# Before Computers
Scytale cipher 
	Sparta 700 BCE  
	Sender and recipient need to share somethings to decryp the message
	How to share this information
Caesar cipher 
	100 BCE
	Few possible key 
	Still need to know the key 
Alphabetic substitution, Poly-alphabetic 
	Vulnerable to frequency attack
Enigma 
	Broken by computer because of limited combinaison $10^{23}$

# Frequency analysis 
- Each language has its own distribution of frequencies of the letters used
- they change from language to language but are public and well-known
### Recipe to obtain this frequency for any language : 

- Take a large sample of text
- Remove spaces, punctuation, and special symbols
- Sort the remaining sequence of letters
- Perform a simple block-based count

# Model

![[2025-09-24-152345_hyprshot.png]]

A Key is long string of random bits 
if the key is $n$ bits long, they are $2^n$ possible keys 
With symmetric encryption, the keys are the same

1. Two parties - Alice and Bob
2. Reliable communication line
3. Shared encryption scheme : $E,D,K_1,K_2$
4. Goal : Send a message $M$ confidentially

## Adversary 
- Passive 
	- Reads the exchanged messages
	- (no change)
- Active 
	- Can modify messages send by Alice or Bob 
	- Can send fake message claiming to be someone else




| Term       | Meaning                                                                                                  |
| ---------- | -------------------------------------------------------------------------------------------------------- |
| plaintext  | information that will be encrypted                                                                       |
| ciphertext | information that has been encrypted, i.e. transformed into incomprehensible text                         |
| key        | sequence of fixed length of bits that appear random; in the symmetric case $K_1 = K_2$                   |
| cipher     | pair of algorithms for encryption and decryption, often denoted as ($E, D$)<br>                          |
| encryptor  | entity that applies a cipher (algorithm $E$), producing a ciphertext<br>                                 |
| decryptor  | entity that applies a cipher (algorithm $D$), producing a plaintext<br>                                  |
| encryption | the operation performed by an encryptor<br>                                                              |
| decryption | the operation performed by a decryptor                                                                   |
| adversary  | entity that attempts to compromise confidentiality, integrity, or availability of<br>information systems |






