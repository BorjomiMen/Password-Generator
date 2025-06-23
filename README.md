# 🔐 Password Generator Application

A secure and feature-rich password generator application with a modern GUI interface, password strength analysis, and history tracking.

## ✨ Features

- **Customizable Password Generation** - Control length and character types
- **Password Strength Analysis** - Visual strength indicators
- **History Tracking** - Save and review generated passwords
- **Export Capabilities** - Save history to CSV files
- **Clipboard Integration** - One-click copy to clipboard
- **Modern GUI** - Themed interface with clean design
- **Persistent History** - JSON storage of password history

## 🚀 Complete Implementation

```python
import tkinter as tk
from tkinter import ttk, filedialog, messagebox
from ttkthemes import ThemedTk
import random
import string
import pyperclip
import json
import os
from datetime import datetime
import pandas as pd

class PasswordGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("🔐 Password Generator")
        self.root.geometry("600x500")
        self.root.resizable(False, False)
        
        self.history = self.load_history()
        self.current_theme = "clearlooks"

        self.create_widgets()
    
    def load_history(self):
        """Load password history from JSON file"""
        try:
            with open("password_history.json", "r") as f:
                return json.load(f)
        except FileNotFoundError:
            return []
    
    def save_history(self):
        """Save password history to JSON file"""
        with open("password_history.json", "w") as f:
            json.dump(self.history, f, indent=2)
    
    def create_widgets(self):
        """Create all GUI components"""
        # Settings frame
        settings_frame = ttk.LabelFrame(self.root, text="Настройки")
        settings_frame.pack(pady=10, padx=10, fill="x")

        ttk.Label(settings_frame, text="Длина пароля:").grid(row=0, column=0)
        self.length_var = tk.IntVar(value=12)
        ttk.Spinbox(settings_frame, from_=8, to=64, 
                    textvariable=self.length_var, width=5).grid(row=0, column=1)

        # Character type checkboxes
        self.upper_var = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Заглавные буквы (A-Z)", 
                        variable=self.upper_var).grid(row=1, column=0, sticky="w")

        self.lower_var = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Строчные буквы (a-z)", 
                        variable=self.lower_var).grid(row=2, column=0, sticky="w")

        self.digits_var = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Цифры (0-9)", 
                        variable=self.digits_var).grid(row=1, column=1, sticky="w")

        self.symbols_var = tk.BooleanVar(value=True)
        ttk.Checkbutton(settings_frame, text="Символы (!, @, #)", 
                        variable=self.symbols_var).grid(row=2, column=1, sticky="w")

        # Button frame
        button_frame = ttk.Frame(self.root)
        button_frame.pack(pady=10)

        self.generate_button = ttk.Button(
            button_frame, 
            text="🎲 Сгенерировать", 
            command=self.generate_password
        )
        self.generate_button.pack(side="left", padx=5)

        self.copy_button = ttk.Button(
            button_frame, 
            text="📋 Скопировать", 
            command=self.copy_to_clipboard
        )
        self.copy_button.pack(side="left", padx=5)

        self.export_button = ttk.Button(
            button_frame, 
            text="📤 Экспорт истории", 
            command=self.export_history
        )
        self.export_button.pack(side="left", padx=5)

        # Result frame
        result_frame = ttk.Frame(self.root)
        result_frame.pack(pady=10, fill="x")

        ttk.Label(result_frame, text="Сгенерированный пароль:").pack()
        self.password_entry = ttk.Entry(
            result_frame, 
            font=("Consolas", 14), 
            justify="center"
        )
        self.password_entry.pack(fill="x", padx=10)

        self.strength_label = ttk.Label(
            result_frame, 
            text="Сложность: ❓", 
            font=("Arial", 10)
        )
        self.strength_label.pack(pady=5)

        # History frame
        history_frame = ttk.LabelFrame(self.root, text="История паролей")
        history_frame.pack(pady=10, padx=10, fill="both", expand=True)

        self.history_list = tk.Listbox(history_frame, height=10)
        self.history_list.pack(fill="both", expand=True, padx=5, pady=5)

        self.update_history_display()

    def generate_password(self):
        """Generate password based on user settings"""
        length = self.length_var.get()
        chars = ""
        
        if self.upper_var.get():
            chars += string.ascii_uppercase
        if self.lower_var.get():
            chars += string.ascii_lowercase
        if self.digits_var.get():
            chars += string.digits
        if self.symbols_var.get():
            chars += "!@#$%^&*()_+-=[]{}|;:,.<>?"

        if not chars:
            messagebox.showwarning("Ошибка", "Выберите хотя бы один тип символов!")
            return

        password = "".join(random.choice(chars) for _ in range(length))
        self.password_entry.delete(0, tk.END)
        self.password_entry.insert(0, password)
        
        strength = self.check_strength(password)
        self.strength_label.config(text=f"Сложность: {strength}")

        # Save to history
        self.history.append({
            "password": password,
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "strength": strength
        })
        self.save_history()
        self.update_history_display()

    def check_strength(self, password):
        """Evaluate password strength"""
        score = 0
        if len(password) >= 12:
            score += 1
        if any(c.islower() for c in password):
            score += 1
        if any(c.isupper() for c in password):
            score += 1
        if any(c.isdigit() for c in password):
            score += 1
        if any(c in "!@#$%^&*()_+-=[]{}|;:,.<>?" for c in password):
            score += 1

        if score <= 2:
            self.strength_label.config(foreground="red")
            return "🔴 Слабый"
        elif score <= 4:
            self.strength_label.config(foreground="orange")
            return "🟡 Средний"
        else:
            self.strength_label.config(foreground="green")
            return "🟢 Сильный"

    def copy_to_clipboard(self):
        """Copy password to clipboard"""
        password = self.password_entry.get()
        if password:
            pyperclip.copy(password)
            messagebox.showinfo("Успех", "Пароль скопирован в буфер обмена!")

    def export_history(self):
        """Export password history to CSV file"""
        filename = filedialog.asksaveasfilename(
            defaultextension=".csv", 
            filetypes=[("CSV", "*.csv")]
        )
        if filename:
            df = pd.DataFrame(self.history)
            df.to_csv(filename, index=False)
            messagebox.showinfo("Успех", f"История сохранена в {filename}")

    def update_history_display(self):
        """Update the history listbox"""
        self.history_list.delete(0, tk.END)
        for entry in reversed(self.history[-10:]): 
            self.history_list.insert(
                tk.END, 
                f"{entry['timestamp']} | {entry['password']} | {entry['strength']}"
            )

if __name__ == "__main__":
    root = ThemedTk(theme="arc")
    app = PasswordGenerator(root)
    root.mainloop()
```
## 📋 Requirements
```bash
📋 Requirements
```
## 🖥️ Usage
1) Run the application:
```bash
python password_generator.py
```
2) Configure password settings:
- Set password length (8-64 characters)
- Select character types to include

3) Click "🎲 Сгенерировать" to generate a password
Use buttons to:
- Copy password to clipboard
- Export password history
- View previous passwords
