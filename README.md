# typing-test_project
import tkinter as tk
import random

sentences = [
    "Typing helps improve your accuracy and speed",
    "Python is a great language to learn and master",
    "Practice daily to enhance your typing performance",
    "The quick brown fox jumps over the lazy dog",
    "Consistency and patience are key to improvement"
]

themes = {
    "Light": {"bg": "white", "fg": "black"},
    "Dark": {"bg": "#1e1e1e", "fg": "white"}
}

time_left = 60
test_running = False
typed_chars = 0
correct_chars = 0
typed_words = 0
current_sentence = ""
shuffled_sentences = []
sentence_index = 0


def shuffle_sentences():
    global shuffled_sentences, sentence_index
    shuffled_sentences = sentences[:]
    random.shuffle(shuffled_sentences)
    sentence_index = 0


def start_test():
    global time_left, test_running, typed_chars, correct_chars, typed_words
    time_left = 60
    typed_chars = correct_chars = typed_words = 0
    test_running = True

    shuffle_sentences()
    entry.config(state=tk.NORMAL)
    entry.delete(0, tk.END)
    entry.focus()

    result_label.config(text="")
    feedback_label.config(text="")

    show_next_sentence()
    update_timer()
    countdown()


def countdown():
    global time_left
    if test_running and time_left > 0:
        time_left -= 1
        update_timer()
        root.after(1000, countdown)
    else:
        finish_test()


def update_timer():
    wpm = round(typed_words / ((60 - time_left) / 60), 2) if (60 - time_left) > 0 else 0
    wpm_label.config(text=f"WPM: {wpm}")
    timer_label.config(text=f"Time Left: {time_left}s")


def show_next_sentence():
    global current_sentence, sentence_index
    if sentence_index >= len(shuffled_sentences):
        shuffle_sentences()

    current_sentence = shuffled_sentences[sentence_index]
    sentence_index += 1

    sentence_label.config(text=current_sentence)
    entry.delete(0, tk.END)


def check_typed(event=None):
    global typed_chars, correct_chars, typed_words

    if not test_running or time_left <= 0:
        return

    typed = entry.get().strip()
    expected = current_sentence.strip()

    if len(typed) >= len(expected):
        typed_chars += len(typed)
        correct_chars += sum(1 for a, b in zip(typed, expected) if a == b)
        typed_words += len(typed.split())
        show_next_sentence()


def finish_test():
    global test_running
    test_running = False
    entry.config(state=tk.DISABLED)

    wpm = round(typed_words, 2)
    accuracy = round((correct_chars / typed_chars) * 100, 2) if typed_chars > 0 else 0

    if wpm < 20:
        feedback = "🔴 Low - Keep practicing!"
        color = "red"
    elif 20 <= wpm <= 40:
        feedback = "🟠 Average - You can do better!"
        color = "orange"
    elif 41 <= wpm <= 60:
        feedback = "🟢 Good - Well done!"
        color = "green"
    else:
        feedback = "🌟 Excellent! You're super fast!"
        color = "blue"

    result_label.config(text=f"WPM: {wpm} | Accuracy: {accuracy}%", fg="black")
    feedback_label.config(text=feedback, fg=color)


def change_theme(choice):
    theme = themes[choice]
    root.config(bg=theme["bg"])
    for widget in root.winfo_children():
        try:
            widget.config(bg=theme["bg"], fg=theme["fg"])
        except:
            pass


root = tk.Tk()
root.title("Typing Speed Test")
root.geometry("800x400")
root.resizable(False, False)

wpm_label = tk.Label(root, text="WPM: 0", font=("Arial", 14, "bold"))
wpm_label.place(x=10, y=10)

timer_label = tk.Label(root, text="Time Left: 60s", font=("Arial", 14, "bold"))
timer_label.place(relx=0.5, y=10, anchor="n")

theme_label = tk.Label(root, text="Theme:", font=("Arial", 12))
theme_label.place(x=680, y=10)

theme_var = tk.StringVar(value="Light")
theme_menu = tk.OptionMenu(root, theme_var, *themes.keys(), command=change_theme)
theme_menu.place(x=740, y=5)

sentence_label = tk.Label(
    root, text="", font=("Arial", 16),
    wraplength=750, justify="center"
)
sentence_label.pack(pady=60)

entry = tk.Entry(root, font=("Arial", 16), width=70, state=tk.DISABLED)
entry.pack(pady=5)
entry.bind("<KeyRelease>", check_typed)

start_btn = tk.Button(
    root, text="Start Typing Test",
    font=("Arial", 12), command=start_test
)
start_btn.pack(pady=10)

result_label = tk.Label(root, text="", font=("Arial", 14))
result_label.pack(pady=5)

feedback_label = tk.Label(root, text="", font=("Arial", 16, "bold"))
feedback_label.pack(pady=5)

change_theme("Light")
root.mainloop()
