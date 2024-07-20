# SOP: Download and Convert O'Reilly Books to Kindle Format

## Index
1. [[#1. Introduction]]
2. [[#2. Prerequisites]]
3. [[#3. Downloading O'Reilly Books]]
4. [[#4. Converting EPUB to Kindle Format]]
5. [[#5. Transferring to Kindle]]
6. [[#6. Troubleshooting]]
7. [[#7. Notes]]
8. [[#8. Links and References]]

## 1. Introduction

This Standard Operating Procedure (SOP) outlines the process of downloading O'Reilly books and converting them to a Kindle-compatible format.

## 2. Prerequisites

Before beginning, ensure you have the following:

- Linux system (commands may need adaptation for other operating systems)
- Docker installed
- Valid O'Reilly subscription
- Calibre installed
- Git installed
- `jq` package installed
- Firefox browser (for cookie extraction)
- USB cable for Kindle connection

To install prerequisites:

```bash
# Install Docker (if not already installed)
sudo apt-get update
sudo apt-get install docker.io

# Install Calibre
sudo -v && wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sudo sh /dev/stdin

# Install jq
sudo apt-get install jq

# Install Git (if not already installed)
sudo apt-get install git
```

## 3. Downloading O'Reilly Books

### 3.1 Set Up O'Reilly Downloader

Clone the O'Reilly downloader repository:

```bash
git clone https://github.com/kirinnee/oreilly-downloader.git
cd oreilly-downloader
```

### 3.2 Extract Cookies from Firefox

1. Log in to [O'Reilly Learning](https://learning.oreilly.com/) using Firefox.
2. Press `F12` to open Developer Tools.
3. Go to the `Network` tab.
4. Navigate to [O'Reilly Profile Page](https://learning.oreilly.com/profile/).
5. Click on the request to `/profile/` in the Network tab.
6. Go to the `Cookies` tab within the request details.
7. Right-click on any value under `Request Cookies` and choose `Copy All`.
8. Paste the copied cookies into a file named `cookie.json`.

### 3.3 Flatten the Cookie JSON

Create and use a script to flatten the JSON structure:

```bash
cat > flatten_cookies.sh << EOL
#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: $0 <input_file>"
    exit 1
fi

input_file="$1"

jq '.["Request Cookies"]' "$input_file" > flattened_cookie.json
EOL

chmod +x flatten_cookies.sh
./flatten_cookies.sh cookie.json
```

### 3.4 Download Books

Use the following command to download books:

```bash
(cat cookie.json | docker run -i kirinnee/orly:latest sso <book_id>) > "<book_title>.epub"
```

Examples:
```bash
(cat cookie.json | docker run -i kirinnee/orly:latest sso 9781491905449) > "MPLS in the SDN Era.epub"
(cat cookie.json | docker run -i kirinnee/orly:latest sso 9780135180471) > "Network Programmability with YANG.epub"
```

## 4. Converting EPUB to Kindle Format

Use Calibre's command-line tool to convert EPUB files to MOBI or AZW3 format:

```bash
ebook-convert "<book_title>.epub" "<book_title>.azw3"
```

Example:
```bash
ebook-convert "MPLS in the SDN Era.epub" "MPLS_in_the_SDN_Era.azw3"
```

## 5. Transferring to Kindle

1. Connect your Kindle to your computer via USB.
2. Copy the converted AZW3 or MOBI file to the "Documents" folder on your Kindle.
3. Safely eject your Kindle.

## 6. Troubleshooting

- If you encounter `libxcb-cursor0` errors when installing Calibre, install it separately:
  ```bash
  sudo apt-get install libxcb-cursor0
  ```
- Ensure all cookies are correctly copied and formatted in the JSON file.
- If downloads fail, check your O'Reilly subscription status and internet connection.

## 7. Notes

- This SOP is designed for Linux environments. Adapt commands as needed for other operating systems.
- Ensure you have the necessary rights to download and convert the books.
- Keep your cookie file secure and do not share it with others.

## 8. Links and References

- [O'Reilly Learning](https://learning.oreilly.com/)
- [Calibre Download Page](https://calibre-ebook.com/download)
- [Docker Installation Guide](https://docs.docker.com/engine/install/)
- [jq Manual](https://stedolan.github.io/jq/manual/)
- [Kindle User Guide](https://www.amazon.com/gp/help/customer/display.html?nodeId=200505520)
