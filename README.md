

# JT/T 808-2023 Binary Frame Visualizer

A modern, single‑page web tool to decode and inspect raw JT/T 808 (2019/2023‑compatible) GPS tracking frames.  
Paste a hex string, hit **Parse frame**, and instantly explore header fields, location data, alarms, status bits, and additional information in a rich UI.

> Built as a browser‑only tool – no backend, no build system.

***

## Features

- **Hex → Structured decode**
  - Accepts hex with or without `7E` frame delimiters.
  - Ignores spaces and line breaks.
  - Validates the XOR checksum.

- **Protocol‑aware parsing**
  - Escape/unescape handling (`7D 02 → 7E`, `7D 01 → 7D`).
  - Message header decoding:
    - Message ID and friendly name.
    - Body properties (length, encryption flags, fragmentation).
    - Terminal phone number (BCD).
    - Message serial number.
    - Fragmented message info (total packets, packet index).

- **Supported message types**
  - `0x0002` – Terminal Heartbeat (no body).
  - `0x0100` – Terminal Registration.
  - `0x0102` – Terminal Authentication.
  - `0x0107` – Query Terminal Attribute Response (basic).
  - `0x0200` – Location Information Report.
  - `0x0704` – Positioning Data Bulk Upload (batch of `0x0200`).
  - `0x8001` – Platform Universal Response.
  - `0x8100` – Terminal Registration Response.

- **0x0200 location details**
  - Alarm flags (32 bits).
  - Status flags (ACC, GNSS constellations, doors, etc.).
  - Latitude / longitude (scaled, degrees).
  - Altitude, speed, direction.
  - BCD time decoding.
  - Additional information items (`0x30`, `0x31`, `0x51`, `0x56`, `0x57`, `0x63`, `0xFD`, …).

- **0x0704 bulk upload**
  - Item count and location type (normal / blind area).
  - Each embedded 0x0200 parsed and shown in its own expandable panel.

- **Polished UI**
  - Dark, dashboard‑style layout.
  - Success/error alerts with meaningful messages.
  - Header summary panel with pills and badges.
  - Collapsible sections for bulk uploads.
  - Hex dump for both body and normalized original frame.

***

## Getting Started

### 1. Clone or download

```bash
git clone https://github.com/rootcastleco/jt808-parser.git
cd your-repo
```


### 2. Open in a browser

You can simply open `index.html` directly, or serve it via a tiny static server:

```bash
# Python 3
python -m http.server 8080

# then open:
# http://localhost:8080/index.html
```

No build step, dependencies, or backend services are required.

***

## Usage

1. Open the HTML page in your browser.
2. Paste a JT/T 808 frame as a hex string into the **Hex String** textarea.
   - You may include or omit starting/ending `7E`.
   - Whitespace is ignored.
3. Click **Parse frame**.
4. Inspect the decoded result:
   - Header (MsgID, phone, serial number, body length, encryption, fragmentation, checksum).
   - Message‑type specific sections (location, registration, responses…).
   - Raw body hex and normalized frame hex.

You can also use the built‑in demo buttons:

- **Load 0x0200 example** – inserts a sample Location Information Report and parses it.
- **Load 0x0704 example** – inserts a bulk upload frame and parses all embedded locations.
- **Clear** – resets input and output.

***

## Project Structure

This project is intentionally minimal:

```text
.
└── index.html   # Contains HTML, CSS and JavaScript for the entire tool
```

All parsing logic, visualization components, and styling live in `index.html`, so you can drop it into any environment that can serve static files.

***

## Implementation Overview

### Frame normalization

- Remove non‑hex characters.
- Optionally strip leading and trailing `7E`.
- Convert pairs of hex characters to bytes.
- Unescape:
  - `0x7D 0x02 → 0x7E`
  - `0x7D 0x01 → 0x7D`

### Checksum

- XOR of all bytes from the start of the header up to the last body byte.
- Compared against the last byte of the unescaped frame.
- Any mismatch is reported as a human‑readable error.

### Header fields

- `msgId` – 2 bytes, big‑endian.
- `bodyProps` – 2 bytes, split into:
  - lower 10 bits: body length.
  - bits 10–12: encryption type.
  - bit 13: fragmentation flag.
- Phone number – 6 bytes BCD → string.
- `serialNumber` – 2 bytes.
- If fragmented:
  - `totalPackets` – 2 bytes.
  - `packetIndex` – 2 bytes.

### Message‑specific decoding

- 0x0200: reads fixed header fields, then loops through additional information TLV items.
- 0x0704: reads count and type, then repeatedly reads length‑prefixed embedded 0x0200 payloads.
- 0x0100 / 0x8100 / 0x0102 / 0x8001: extract fields according to the protocol and display them in the UI.

***

## Customization

You can easily extend or adapt the parser:

- Add new message IDs and names to the `MSG_NAMES` map.
- Implement additional parsers and hook them into the `switch (frame.msgId)` block.
- Extend `ADDITIONAL_INFO_IDS` to cover more custom or vendor‑specific items.
- Adjust the UI to match your branding or integrate into your own debug tools.

***

## Roadmap Ideas

- Export parsed data as JSON.
- Import/export frames from text files.
- Add more message types (e.g., media, events, driver info).
- Add unit tests for parsing functions.
- Optional multi‑language UI (English, Chinese, Turkish).

***

## License

Choose a license that matches your needs, for example:

```text
MIT License

Copyright (c) 
