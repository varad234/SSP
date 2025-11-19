# SSP
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
import os

# ---------- AES Key ----------
# Use the AES key you generated
key = os.urandom(32)  # 32 bytes = 256-bit AES key
print("AES Key (hex):", key.hex())

# ---------- Initialization Vector ----------
# IV should be random and 16 bytes for AES
iv = os.urandom(16)

# ---------- Read plaintext from file ----------
with open("sample.txt", "rb") as f:
    plaintext = f.read()

# ---------- Encrypt ----------
cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
encryptor = cipher.encryptor()

# AES requires plaintext to be multiple of block size (16 bytes)
from cryptography.hazmat.primitives import padding
padder = padding.PKCS7(128).padder()
padded_plaintext = padder.update(plaintext) + padder.finalize()

ciphertext = encryptor.update(padded_plaintext) + encryptor.finalize()

# Save ciphertext to a file
with open("encrypted_file.bin", "wb") as f:
    f.write(iv + ciphertext)  # prepend IV for decryption

print("Encryption complete! Saved as 'encrypted_file.bin'")

# ---------- Decrypt ----------
# Read the encrypted file
with open("encrypted_file.bin", "rb") as f:
    file_data = f.read()

# Extract IV and ciphertext
iv_from_file = file_data[:16]
ciphertext_from_file = file_data[16:]

cipher = Cipher(algorithms.AES(key), modes.CBC(iv_from_file), backend=default_backend())
decryptor = cipher.decryptor()
padded_plaintext = decryptor.update(ciphertext_from_file) + decryptor.finalize()

# Unpad plaintext
unpadder = padding.PKCS7(128).unpadder()
decrypted_text = unpadder.update(padded_plaintext) + unpadder.finalize()

# Save decrypted text to a new file
with open("decrypted_file.txt", "wb") as f:
    f.write(decrypted_text)

print("Decryption complete! Saved as 'decrypted_file.txt'")
