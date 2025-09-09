import subprocess
import sys
from tkinter import *
from tkinter import scrolledtext, messagebox
import threading

StartingPoint = " Tung Chung Catholic School (Secondary Section)"


def install_package(package):
    subprocess.check_call([sys.executable, "-m", "pip", "install", package])


# Ensure OpenAI package is installed
try:
    from openai import OpenAI
except ImportError:
    print("Installing OpenAI package...")
    install_package("openai")
    from openai import OpenAI


class AITravelAssistant:
    def __init__(self):
        self.window = Tk()
        self.window.title("AI travel assistant")
        self.window.geometry("600x500")

        self.font = ("arial",12)
        self.title_font = ("arial",16, "bold")

        # Initialize OpenAI client
        self.client = OpenAI(
            api_key="sk-2cea37d74eaf407bb2a769ec386bb1e0",
            base_url="https://api.deepseek.com"
        )

        self.setup_gui()

    def setup_gui(self):
        # Title
        title_label = Label(self.window, text="AI travel assistant", font=self.title_font)
        title_label.pack(pady=10)

        # Starting point info
        start_label = Label(self.window, text=f"Starting point:{StartingPoint}", font=self.font)
        start_label.pack(pady=5)

        # Instructions
        instruction_text = "Enter your destination and click 'search' to get the best route to get there by MTR, Bus, or walking."
        instruction_label = Label(self.window, text=instruction_text, font=self.font, wraplength=500)
        instruction_label.pack(pady=5)

        # Chat display area
        chat_frame = Frame(self.window)
        chat_frame.pack(fill=BOTH, expand=True, padx=10, pady=10)

        self.chat_display = scrolledtext.ScrolledText(
            chat_frame,
            font=self.font,
            wrap=WORD,
            state=DISABLED,
            height=15
        )
        self.chat_display.pack(fill=BOTH, expand=True)

        # Input frame
        input_frame = Frame(self.window)
        input_frame.pack(fill=X, padx=10, pady=5)

        # Destination input
        Label(input_frame, text="Destination:", font=self.font).pack(side=LEFT)

        self.destination_entry = Entry(input_frame, font=self.font, width=30)
        self.destination_entry.pack(side=LEFT, fill=X, expand=True, padx=5)
        self.destination_entry.bind("<Return>", lambda event: self.send_message())

        # Send button
        self.send_button = Button(
            input_frame,
            text="Search",
            font=self.font,
            command=self.send_message,
            bg="#4CAF50",
            fg="white"
        )
        self.send_button.pack(side=RIGHT, padx=5)

        # Add welcome message
        self.add_message("System", "Welcome to AI travel assistant, please enter your destination", "#666666")

    def add_message(self, sender, message, color="#000000"):
        self.chat_display.config(state=NORMAL)
        self.chat_display.insert(END, f"{sender}: ", "sender")
        self.chat_display.insert(END, f"{message}\n\n", "message")
        self.chat_display.tag_config("sender", foreground="#0066CC", font=(self.font[0], self.font[1], "bold"))
        self.chat_display.tag_config("message", foreground=color, font=self.font)
        self.chat_display.config(state=DISABLED)


    def send_message(self):
        destination = self.destination_entry.get().strip()
        if not destination:
            return

        if destination.lower() in ['quit', 'exit', 'q', '退出', '結束']:
            self.window.quit()
            return

        # Clear input
        self.destination_entry.delete(0, END)

        # Add user message
        self.add_message("You", destination, "#0066CC")

        # Disable send button while processing
        self.send_button.config(state=DISABLED, text="Searching...")

        # Get AI response in a separate thread to avoid freezing GUI
        threading.Thread(target=self.get_ai_response, args=(destination,), daemon=True).start()

    def get_ai_response(self, destination):
        try:
            response = self.client.chat.completions.create(
                model="deepseek-chat",
                messages=[
                    {"role": "system",
                     "content": f"You are a local helpful assistant in Hong Kong, the starting point of the journey would always be {StartingPoint}. I will tell you my destination and you will give me the best route to get there by MTR, Bus, or even walking. Not driving and not taxi. You will also give me the estimated time to get there. You will also give me the estimated cost of the journey. You will always explain the route in detail in English."},
                    {"role": "user", "content": destination}
                ],
                stream=False
            )

            ai_message = response.choices[0].message.content

            # Update GUI in main thread
            self.window.after(0, lambda: self.handle_ai_response(ai_message))

        except Exception as e:
            error_message = f"錯誤：{str(e)}"
            self.window.after(0, lambda: self.handle_ai_response(error_message, is_error=True))

    def handle_ai_response(self, message, is_error=False):
        color = "#FF0000" if is_error else "#008000"
        sender = "錯誤" if is_error else "AI assistant"

        self.add_message(sender, message, color)

        # Re-enable send button
        self.send_button.config(state=NORMAL, text="Search")

    def run(self):
        self.window.mainloop()


if __name__ == "__main__":
    app = AITravelAssistant()
    app.run()
