# Aditya-AI-Assistant-Empowering-Efficiency

import tkinter as tk
from tkinter import messagebox
import pyttsx3
import speech_recognition as sr
import datetime
import wikipedia
import pyjokes
import pywhatkit
import threading
import re
import math

class Aditya:
    def __init__(self):
        self.engine = pyttsx3.init()
        voices = self.engine.getProperty('voices')
        self.engine.setProperty('voice', voices[1].id)
        self.engine.setProperty("rate", 150)

    def speak(self, text):
        print(f"Aditya: {text}")
        self.engine.say(text)
        self.engine.runAndWait()

    def listen(self):
        r = sr.Recognizer()
        with sr.Microphone() as source:
            self.speak("Listening...")
            r.adjust_for_ambient_noise(source, duration=1)
            audio = r.listen(source, timeout=5, phrase_time_limit=7)
        try:
            return r.recognize_google(audio)
        except sr.UnknownValueError:
            return "Sorry, I didn't catch that."
        except sr.RequestError:
            return "Network error. Please check your connection."

    def handle(self, command):
        command = command.lower()
        if "play song" in command or "play" in command:
            song = command.replace("play", "").strip()
            pywhatkit.playonyt(song)
            return f"Playing {song} on YouTube."
        elif "time" in command:
            return f"The current time is {datetime.datetime.now().strftime('%I:%M %p')}."
        elif "search" in command:
            query = command.replace("search", "").strip()
            pywhatkit.search(query)
            return f"Searching for {query} on Chrome."
        elif "joke" in command:
            return pyjokes.get_joke()
        elif "wikipedia" in command:
            topic = command.replace("wikipedia", "").strip()
            try:
                return wikipedia.summary(topic, sentences=2)
            except:
                return "No Wikipedia results found."
        elif any(op in command for op in ["plus", "minus", "times", "divided", "into", "power", "square root", "sqrt", "+", "-", "*", "/"]):
            try:
                return self.solve_math(command)
            except Exception:
                return "Sorry, I couldn't understand the math problem."
        elif "your name" in command:
            return "My name is Aditya, your AI assistant."
        elif "exit" in command or "quit" in command:
            return "exit"
        else:
            return "I don't recognize that command yet."

    def solve_math(self, command):
        command = command.lower()
        command = command.replace("plus", "+").replace("minus", "-").replace("times", "*") \
                         .replace("multiplied by", "*").replace("x", "*").replace("into", "*") \
                         .replace("divided by", "/").replace("over", "/").replace("power", "**") \
                         .replace("square root of", "math.sqrt").replace("sqrt", "math.sqrt")

        expression = re.sub(r"[^0-9\+\-\*/\.\(\)mathsqrt]", "", command)
        result = eval(expression, {"__builtins__": None, "math": math})
        return f"The answer is {result}"

class AdityaGUI:
    def __init__(self):
        self.bot = Aditya()
        self.root = tk.Tk()
        self.root.title("Aditya - AI Assistant")
        self.root.geometry("950x750")
        self.root.configure(bg="#1e1e2f")

        self.title = tk.Label(self.root, text="✨ Aditya - Your AI Assistant", font=("Segoe UI", 26, "bold"), fg="#00fff5", bg="#1e1e2f")
        self.title.pack(pady=20)

        self.output = tk.Text(self.root, height=14, bg="#282a36", fg="#f8f8f2", font=("Consolas", 13), bd=0, relief=tk.FLAT, wrap=tk.WORD)
        self.output.pack(padx=30, pady=10, fill=tk.BOTH, expand=True)

        self.frame = tk.Frame(self.root, bg="#1e1e2f")
        self.frame.pack(pady=10)

        self.speak_btn = tk.Button(self.frame, text="🎤 Speak", font=("Segoe UI", 14), bg="#00fff5", fg="#1e1e2f",
                                   relief="raised", bd=4, width=12, height=2, command=self.start_listening)
        self.speak_btn.grid(row=0, column=0, padx=10)

        self.exit_btn = tk.Button(self.frame, text="❌ Exit", font=("Segoe UI", 14), bg="#ff005c", fg="white",
                                  relief="raised", bd=4, width=12, height=2, command=self.exit_app)
        self.exit_btn.grid(row=0, column=1, padx=10)

        self.entry_frame = tk.Frame(self.root, bg="#1e1e2f")
        self.entry_frame.pack(pady=10)

        self.command_entry = tk.Entry(self.entry_frame, font=("Consolas", 14), width=50, bd=4)
        self.command_entry.grid(row=0, column=0, padx=10)

        self.submit_btn = tk.Button(self.entry_frame, text="📨 Submit", font=("Segoe UI", 12), bg="#00ff88", fg="#1e1e2f",
                                    relief="raised", bd=3, command=self.handle_text_command)
        self.submit_btn.grid(row=0, column=1)

        self.bot.speak("Hello, I am Aditya. How can I help you today?")
        self.display("Aditya", "Hello, I am Aditya. How can I help you today?")

    def start_listening(self):
        threading.Thread(target=self.listen_and_respond).start()

    def listen_and_respond(self):
        self.display("System", "Listening...")
        command = self.bot.listen()
        self.display("You", command)
        response = self.bot.handle(command)
        if response == "exit":
            self.exit_app()
        else:
            self.display("Aditya", response)
            self.bot.speak(response)

    def handle_text_command(self):
        command = self.command_entry.get()
        self.command_entry.delete(0, tk.END)
        if command.strip() == "":
            return
        self.display("You", command)
        response = self.bot.handle(command)
        if response == "exit":
            self.exit_app()
        else:
            self.display("Aditya", response)
            self.bot.speak(response)

    def display(self, who, msg):
        self.output.insert(tk.END, f"{who}: {msg}\n\n")
        self.output.see(tk.END)

    def exit_app(self):
        if messagebox.askyesno("Exit", "Are you sure you want to exit?"):
            self.root.destroy()

    def run(self):
        self.root.mainloop()

if __name__ == "__main__":
    ui = AdityaGUI()
    ui.run()
