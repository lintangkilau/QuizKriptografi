﻿import tkinter as tk
from tkinter import filedialog, messagebox
import numpy as np

# Vigenère Cipher
def vigenere_encrypt(plaintext, key):
    ciphertext = ""
    key = key.upper()
    key_length = len(key)
    for i, char in enumerate(plaintext):
        if char.isalpha():
            shift = ord(key[i % key_length]) - ord('A')
            if char.isupper():
                ciphertext += chr((ord(char) - ord('A') + shift) % 26 + ord('A'))
            else:
                ciphertext += chr((ord(char) - ord('a') + shift) % 26 + ord('a'))
        else:
            ciphertext += char
    return ciphertext

def vigenere_decrypt(ciphertext, key):
    plaintext = ""
    key = key.upper()
    key_length = len(key)
    for i, char in enumerate(ciphertext):
        if char.isalpha():
            shift = ord(key[i % key_length]) - ord('A')
            if char.isupper():
                plaintext += chr((ord(char) - ord('A') - shift) % 26 + ord('A'))
            else:
                plaintext += chr((ord(char) - ord('a') - shift) % 26 + ord('a'))
        else:
            plaintext += char
    return plaintext

# Playfair Cipher (5x5 matrix)
def generate_playfair_matrix(key):
    alphabet = "ABCDEFGHIKLMNOPQRSTUVWXYZ"
    matrix = []
    for char in key.upper():
        if char not in matrix and char != 'J':
            matrix.append(char)
    for char in alphabet:
        if char not in matrix:
            matrix.append(char)
    return np.array(matrix).reshape(5, 5)

def playfair_encrypt(plaintext, key):
    matrix = generate_playfair_matrix(key)
    plaintext = plaintext.upper().replace("J", "I").replace(" ", "")
    if len(plaintext) % 2 != 0:
        plaintext += "X"
    ciphertext = ""
    for i in range(0, len(plaintext), 2):
        a, b = plaintext[i], plaintext[i+1]
        row_a, col_a = np.where(matrix == a)
        row_b, col_b = np.where(matrix == b)
        if row_a == row_b:
            ciphertext += matrix[row_a, (col_a + 1) % 5] + matrix[row_b, (col_b + 1) % 5]
        elif col_a == col_b:
            ciphertext += matrix[(row_a + 1) % 5, col_a] + matrix[(row_b + 1) % 5, col_b]
        else:
            ciphertext += matrix[row_a, col_b] + matrix[row_b, col_a]
    return ciphertext

def playfair_decrypt(ciphertext, key):
    matrix = generate_playfair_matrix(key)
    plaintext = ""
    for i in range(0, len(ciphertext), 2):
        a, b = ciphertext[i], ciphertext[i+1]
        row_a, col_a = np.where(matrix == a)
        row_b, col_b = np.where(matrix == b)
        if row_a == row_b:
            plaintext += matrix[row_a, (col_a - 1) % 5] + matrix[row_b, (col_b - 1) % 5]
        elif col_a == col_b:
            plaintext += matrix[(row_a - 1) % 5, col_a] + matrix[(row_b - 1) % 5, col_b]
        else:
            plaintext += matrix[row_a, col_b] + matrix[row_b, col_a]
    return plaintext

# Hill Cipher (2x2 matrix for simplicity)
def hill_encrypt(plaintext, key):
    plaintext = plaintext.upper().replace(" ", "")
    if len(plaintext) % 2 != 0:
        plaintext += "X"
    matrix_key = np.array([[key[0], key[1]], [key[2], key[3]]])
    ciphertext = ""
    for i in range(0, len(plaintext), 2):
        vector = np.array([[ord(plaintext[i]) - 65], [ord(plaintext[i+1]) - 65]])
        result = np.dot(matrix_key, vector) % 26
        ciphertext += chr(result[0][0] + 65) + chr(result[1][0] + 65)
    return ciphertext

def hill_decrypt(ciphertext, key):
    matrix_key = np.array([[key[0], key[1]], [key[2], key[3]]])
    determinant = np.linalg.det(matrix_key) % 26
    inverse_matrix = np.linalg.inv(matrix_key).astype(int) % 26
    plaintext = ""
    for i in range(0, len(ciphertext), 2):
        vector = np.array([[ord(ciphertext[i]) - 65], [ord(ciphertext[i+1]) - 65]])
        result = np.dot(inverse_matrix, vector) % 26
        plaintext += chr(result[0][0] + 65) + chr(result[1][0] + 65)
    return plaintext

# GUI setup using tkinter
def create_gui():
    root = tk.Tk()
    root.title("Cipher Program")
    # Functionality for file upload
    def upload_file():
        file_path = filedialog.askopenfilename(filetypes=[("Text files", "*.txt")])
        if file_path:
            with open(file_path, 'r') as file:
                message_input.delete(1.0, tk.END)
                message_input.insert(tk.END, file.read())

    # Functionality for encryption and decryption
    def process_cipher(operation):
        message = message_input.get(1.0, tk.END).strip()
        key = key_input.get().strip()
        cipher_type = cipher_var.get()
        
        if len(key) < 12:
            messagebox.showwarning("Invalid Key", "The key must be at least 12 characters long!")
            return

        try:
            if cipher_type == "Vigenere":
                if operation == "encrypt":
                    result = vigenere_encrypt(message, key)
                else:
                    result = vigenere_decrypt(message, key)
            elif cipher_type == "Playfair":
                if operation == "encrypt":
                    result = playfair_encrypt(message, key)
                else:
                    result = playfair_decrypt(message, key)
            elif cipher_type == "Hill":
                # For Hill Cipher, key must be a 2x2 matrix
                key_matrix = [ord(c.upper()) - 65 for c in key[:4]]
                if operation == "encrypt":
                    result = hill_encrypt(message, key_matrix)
                else:
                    result = hill_decrypt(message, key_matrix)
            output_text.delete(1.0, tk.END)
            output_text.insert(tk.END, result)
        except Exception as e:
            messagebox.showerror("Error", f"An error occurred: {e}")

    # GUI layout
    tk.Label(root, text="Message Input:").pack()
    message_input = tk.Text(root, height=10, width=50)
    message_input.pack()

    tk.Button(root, text="Upload .txt File", command=upload_file).pack()

    tk.Label(root, text="Key (min 12 chars):").pack()
    key_input = tk.Entry(root, width=50)
    key_input.pack()

    tk.Label(root, text="Choose Cipher:").pack()
    cipher_var = tk.StringVar(value="Vigenere")
    tk.Radiobutton(root, text="Vigenere", variable=cipher_var, value="Vigenere").pack()
    tk.Radiobutton(root, text="Playfair", variable=cipher_var, value="Playfair").pack()
    tk.Radiobutton(root, text="Hill", variable=cipher_var, value="Hill").pack()

    tk.Button(root, text="Encrypt", command=lambda: process_cipher("encrypt")).pack()
    tk.Button(root, text="Decrypt", command=lambda: process_cipher("decrypt")).pack()

    tk.Label(root, text="Output:").pack()
    output_text = tk.Text(root, height=10, width=50)
    output_text.pack()

    root.mainloop()

# Run the GUI
if __name__ == "__main__":
    create_gui()
    
