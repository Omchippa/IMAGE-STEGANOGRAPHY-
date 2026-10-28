# IMAGE-STEGANOGRAPHY-
Command-line image steganography tool in C using the LSB technique.
# 🖼️ Image Steganography in C (LSB Technique)

A command-line steganography tool written in C that hides a secret text file inside a BMP image by modifying the least significant bits (LSB) of the pixel data. The changes are invisible to the human eye, and the hidden file can be recovered later using a magic string as a passcode.

---

## ✨ Features

- **Encode** a secret text file into a 24-bit BMP image
- **Decode** the hidden file back out of the stego image
- **Magic string** protection: decoding only proceeds if the correct string is entered
- **Capacity check** before encoding, so the image is never corrupted by an oversized payload
- **File extension preserved**: the original extension is stored in the image and restored on decode
- **Custom output names** for both the stego image and the decoded file
- **Step-by-step status messages** for every stage of encoding and decoding

---

## 📁 Project Structure

```
steganography/
├── main.c         # Entry point, argument handling, operation selection
├── encode.c       # Encoding logic (LSB embedding)
├── encode.h
├── decode.c       # Decoding logic (LSB extraction)
├── decode.h
├── common.h       # Shared constants (default magic string)
├── types.h        # Status and operation type definitions
├── beautiful.bmp  # Sample source image
└── secret.txt     # Sample secret file
```

---

## 🧠 How It Works

Every pixel in a 24-bit BMP is stored as 3 bytes (blue, green, red). Changing the lowest bit of a byte alters its value by at most 1, which is not visible to the eye. This project uses that fact to store one bit of secret data in each image byte.

**Example:** hiding the character `A` (`01000001`) in 8 image bytes

```
Image bytes (before): 10110100 11001011 01101110 10010010 11100101 00111000 10101100 01110011
Secret bits         :        0        1        0        0        0        0        0        1
Image bytes (after) : 10110100 11001011 01101110 10010010 11100100 00111000 10101100 01110011
                                                                  ^ only the last bit of each byte changes
```

### Data Layout Inside the Stego Image

The first 54 bytes (the BMP header) are copied unchanged. Everything after it is embedded in this order:

| Order | Data | Size in Image |
|---|---|---|
| 1 | Magic string | 8 bytes per character |
| 2 | Secret file extension size | 32 bytes (32 bits) |
| 3 | Secret file extension (e.g. `.txt`) | 8 bytes per character |
| 4 | Secret file size | 32 bytes (32 bits) |
| 5 | Secret file data | 8 bytes per character |
| 6 | Remaining image data | Copied unchanged |

### Capacity

The image must be large enough to hold everything above. Since each secret byte needs 8 image bytes, the usable payload is roughly one eighth of the image's pixel data. For example, a 1024 × 768 BMP can hide about 290 KB.

---

## 🚀 Getting Started

### Prerequisites

A C compiler such as GCC.

### Build

```bash
gcc main.c encode.c decode.c -o stego
```

---

## 🖥️ Usage

### Encoding

```bash
./stego -e <source_image.bmp> <secret_file.txt> [output_stego.bmp]
```

| Argument | Description |
|---|---|
| `-e` | Select encode mode |
| `source_image.bmp` | The BMP image to hide data in |
| `secret_file.txt` | The text file to hide |
| `output_stego.bmp` | Optional. Defaults to `stego.bmp` |

**Example:**

```bash
./stego -e beautiful.bmp secret.txt
```

You will be prompted to enter a magic string. Remember it, because it is required to decode.

### Decoding

```bash
./stego -d <stego_image.bmp> [output_file_name]
```

| Argument | Description |
|---|---|
| `-d` | Select decode mode |
| `stego_image.bmp` | The image containing hidden data |
| `output_file_name` | Optional. Defaults to `output.txt`. If given, the original extension is added automatically |

**Example:**

```bash
./stego -d stego.bmp
./stego -d stego.bmp decoded
```

You will be prompted for the same magic string used during encoding.

### Sample Session

```
$ ./stego -e beautiful.bmp secret.txt
You choosed encoding
BMP file is present
Text file is present
Read and validate successful
<------- Started Encoding ------->
Source file opened successfully
Secret file opened successfully
Stego file opened successfully
Files opened successfully
Enter magic string: #*
Capacity checked successfully
Header copied successfully
Magic string encoded successfully
Extension size encoded successfully
Secret file extension encoded successfully
Secret file size encoded successfully
Secret data encoded successfully
Remaining image data copied successfully
Encoding successful
```

---

## ⚠️ Limitations

- Works only with **24-bit uncompressed BMP** images (a 54-byte header is assumed). JPG and PNG are not supported.
- The secret file must have a `.txt` extension when encoding.
- The magic string can be at most 9 characters when decoding.
- The magic string and data are not encrypted. This is hiding, not encryption.
- The stego image must not be recompressed or converted to another format, or the hidden data will be lost.

---

## 🔮 Future Improvements

- [ ] Support any file type as the secret payload (images, PDFs, binaries)
- [ ] Encrypt the secret data before embedding
- [ ] Support PNG images
- [ ] Add a `--help` option and stricter input validation
- [ ] Report image capacity before encoding

---

## 🧩 Concepts Demonstrated

- Bitwise operations (masking, shifting, setting and clearing bits)
- Binary file handling with `fopen`, `fread`, `fwrite`, `fseek`
- BMP file format and header structure
- Command-line argument parsing
- Modular programming with multiple source and header files
- Structures, enums, and function return status codes
## ▶️ How to Run

### Step 1: Compile

```bash
gcc main_1.c encode.c decode.c -o stego
```

### Step 2: Encode (hide the secret file)

```bash
./stego -e beautiful.bmp secret.txt stego.bmp
```

| Argument | Meaning |
|---|---|
| `-e` | Encode mode |
| `beautiful.bmp` | Source image (must be a 24-bit BMP) |
| `secret.txt` | Text file to hide |
| `stego.bmp` | Output image (optional, defaults to `stego.bmp`) |

When prompted, enter a magic string (for example `#*`). **Remember it, because you need it to decode.**

**Expected output:**

```
You choosed encoding
BMP file is present
Text file is present
Stego image file is present
Read and validate successful
<------- Started Encoding ------->
Source file opened successfully
Secret file opened successfully
Stego file opened successfully
Files opened successfully
Enter magic string: #*
Capacity checked successfully
Header copied successfully
Magic string encoded successfully
Extension size encoded successfully
Secret file extension encoded successfully
Secret file size encoded successfully
Secret data encoded successfully
Remaining image data copied successfully
Encoding successful
```

### Step 3: Decode (extract the secret file)

```bash
./stego -d stego.bmp output
```

| Argument | Meaning |
|---|---|
| `-d` | Decode mode |
| `stego.bmp` | Image containing the hidden data |
| `output` | Output file name (optional). The original extension is added automatically, so this creates `output.txt`. If omitted, the file is saved as `output.txt` |

When prompted, enter the **same magic string** used during encoding.

**Expected output:**

```
You choosed decoding
.bmp file present
destination file is present
Read and validate successful
<------ Started Decoding ------->
Stego image file opened successfully
Files opened successfully
Enter the magic string: #*
Magic string decoded successfully
User magic     : #*
Decoded magic  : #*
File extension size decoded successfully
File extension size = 4 bytes
File extension decoded successfully
File extension is .txt
Output file name is output.txt
Secret file size decoded successfully
Secret data size is 20
Secret data decoded successfully
Decoding successful
```

### Step 4: Verify

Check that the decoded file matches the original:

```bash
# Linux / macOS
cat output.txt
diff secret.txt output.txt

# Windows
type output.txt
fc secret.txt output.txt
```

If the magic string is wrong, decoding stops with `Magic string does not match` and no file is produced.

### Quick Reference

| Task | Command |
|---|---|
| Encode | `./stego -e beautiful.bmp secret.txt stego.bmp` |
| Decode | `./stego -d stego.bmp output` |
| Run on Windows | Use `stego.exe` in place of `./stego` |

