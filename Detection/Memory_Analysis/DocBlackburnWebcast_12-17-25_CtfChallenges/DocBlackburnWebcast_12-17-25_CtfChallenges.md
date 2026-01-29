<a name="_91bto8ggi6kz"></a>**Challenge 1: The Raw Packet (Binary)**

**Context:** You are manually inspecting a payload from a suspicious UDP packet captured on a legacy ICS (industrial control system) network. The protocol is proprietary, so Wireshark cannot decode it and displays only the raw bit stream.

**Question:** Translate the raw binary data to reveal the signature hidden in the packet payload.

**Data:**

01000110 01001100 01000001 01000111 01111011 01000010 01001001 01001110 00110010 01000001 01010011 01000011 01111101

**How to solve:**

1. **Split:** The data is already grouped into 8-bit chunks (bytes).
1. **Convert:** Turn each 8-bit binary chunk into a Decimal number.
1. **Map:** Look up the Decimal number in the ASCII table.
   1. 01000110 → 70 → **F**
   1. 01001100 → 76 → **L**
   1. (Continue for all bytes...)

**Flag:** FLAG{BIN2ASC}

![ref1]

<a name="_385fjxco0gsk"></a>**Challenge 2: The Memory Dump (Hex & Endianness)**

**Context:** You are performing forensics on a malware sample. You have extracted a hex string from the binary's active memory, but because the malware was running on a 32-bit Little Endian architecture, the string appears scrambled in the hex editor.

**Question:** Reassemble the 4-byte blocks into the correct order to reveal the hidden configuration key.

**Data:**

47414c465845487b444e455f7d4e4149

**How to solve:**

1. **Group:** Split the hex string into 4-byte blocks (8 hex characters per block).
   1. 47414c46 | 5845487b | 444e455f | 7d4e4149
1. **Reverse:** The system reads bytes "backwards" inside each block. Reverse the order of the bytes (pairs) in each block.
   1. 47 41 4c 46 becomes 46 4c 41 47 (ASCII: **FLAG**)
   1. 58 45 48 7b becomes 7b 48 45 58 (ASCII: **{HEX**)
   1. 44 4e 45 5f becomes 5f 45 4e 44 (ASCII: **\_END**)
   1. 7d 4e 41 49 becomes 49 41 4e 7d (ASCII: **IAN}**)

**Flag:** FLAG{HEX\_ENDIAN}

![ref1]

<a name="_fk7mvtsr34o0"></a>**Challenge 3: The Obfuscated Script (Decimal Shift)**

**Context:** During an incident response engagement, you discovered a PowerShell script containing a hardcoded array of numbers. It appears the attacker used a simple "Caesar cipher" style shift to hide the command and evade antivirus detection.

**Question:** Determine the numeric offset used to obfuscate the message and decode the flag.

**Data:**

77, 83, 72, 78, 130, 75, 76, 74, 102, 90, 79, 80, 77, 91, 132

**How to solve:**

1. **Analyze:** The first number is 77 (ASCII 'M'). We expect the flag to start with 70 (ASCII 'F').
1. **Calculate Offset:** 77−70=7. The shift is +7.
1. **Decrypt:** Subtract 7 from every number in the list.
   1. 77 - 7 = 70 (F)
   1. 83 - 7 = 76 (L)
   1. 130 - 7 = 123 ({)
1. **Convert:** Translate the new decimal values to ASCII.

**Flag:** FLAG{DEC\_SHIFT}

![ref1]

<a name="_gigp21pl32t0"></a>**Challenge 4: The Database Token (Big Integer)**

**Context:** You are auditing a web application database and find a "SessionID" field. Instead of a standard text string, the database stores the value as one massive integer to optimize indexing performance.

**Question:** Convert this large integer back into its original text representation to see what the session token actually says.

**Data:**

23921125961261215675282957964529231025021

**How to solve:**

1. **Understand:** The computer sees the whole text string as one giant number.
1. **Convert to Hex:** Use a calculator (or Python) to convert the decimal integer to Hexadecimal.
   1. Result: 464c41477b4d495845445f42415345537d
1. **Split:** Break the hex result into bytes (pairs of two).
   1. 46 4c 41 47 ...
1. **Read:** Convert the hex pairs to ASCII characters.
   1. 46 = **F**
   1. 4c = **L**
   1. 41 = **A**
   1. 47 = **G**

**Flag:** FLAG{MIXED\_BASES}

[ref1]: Aspose.Words.0c50e5d3-ca3a-473e-a059-91ca05283ba2.001.png
