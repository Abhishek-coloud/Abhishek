# Abhishek
import speech_recognition as sr
import pyttsx3
import os
import webbrowser
from datetime import datetime
import yt_dlp as youtube_dl
from fuzzywuzzy import process
import time

# --- Setup TTS Engine ---
engine = pyttsx3.init()

def choose_voice():
    voices = engine.getProperty('voices')
    print("Who would you like to be your assistant? Male or Female?")
    choice = input("Type 'male' or 'female': ").lower()

    # List available voices
    print("Available voices:")
    for voice in voices:
        print(voice.name)

    male_voice = None
    female_voice = None
    
    for voice in voices:
        if "male" in voice.name.lower():
            male_voice = voice.id
        elif "female" in voice.name.lower():
            female_voice = voice.id

    if "female" in choice and female_voice:
        engine.setProperty('voice', female_voice)
    elif "male" in choice and male_voice:
        engine.setProperty('voice', male_voice)
    else:
        print("Couldn't find requested voice, setting to default.")
        engine.setProperty('voice', voices[0].id)

    engine.setProperty('rate', 150)
    engine.setProperty('volume', 0.9)

choose_voice()

# --- Assistant Wake Word ---
WAKE_WORD = "hey buddy"

# --- List of Known Commands ---
commands_list = [
    "open google", "open youtube", "open github", "open facebook",
    "open twitter", "open instagram", "open linkedin", "play music on youtube",
    "open chatgpt", "open dalle", "open openai", "open notepad",
    "what is your name", "exit", "quit", "stop", "shutdown"
]

# --- Functions ---

def speak(text):
    """Speak the given text."""
    engine.say(text)
    engine.runAndWait()

def greet_user():
    """Greet based on time."""
    hour = datetime.now().hour
    if hour < 12:
        speak("Good Morning!")
    elif 12 <= hour < 18:
        speak("Good Afternoon!")
    else:
        speak("Good Evening!")

def listen(timeout=5):
    """Listen to user."""
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        try:
            audio = recognizer.listen(source, timeout=timeout)
            command = recognizer.recognize_google(audio)
            print(f"You said: {command}")
            return command.lower()
        except (sr.UnknownValueError, sr.RequestError, sr.WaitTimeoutError):
            return ""

def open_web_page(site):
    """Open a web page."""
    sites = {
        "google": "https://www.google.com",
        "youtube": "https://www.youtube.com",
        "github": "https://www.github.com",
        "facebook": "https://www.facebook.com",
        "twitter": "https://www.twitter.com",
        "instagram": "https://www.instagram.com",
        "linkedin": "https://www.linkedin.com",
        "chatgpt": "https://chat.openai.com",
        "dalle": "https://openai.com/dall-e",
        "openai": "https://openai.com"
    }
    if site in sites:
        speak(f"Opening {site.capitalize()}.")
        webbrowser.open(sites[site])
    else:
        speak("Sorry, I cannot open that website.")

def download_song(song_name):
    """Download a song from YouTube."""
    speak(f"Downloading {song_name} from YouTube.")
    ydl_opts = {
        'format': 'bestaudio/best',
        'outtmpl': f'{song_name}.mp3',
        'postprocessors': [{
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'mp3',
            'preferredquality': '192',
        }],
        'quiet': True
    }
    with youtube_dl.YoutubeDL(ydl_opts) as ydl:
        search_url = f"ytsearch:{song_name}"
        ydl.download([search_url])
    speak(f"Downloaded {song_name} successfully.")

def execute_command(command):
    """Execute user command."""
    if "open" in command:
        for site in ["google", "youtube", "github", "facebook", "twitter", "instagram", "linkedin", "chatgpt", "dalle", "openai"]:
            if site in command:
                open_web_page(site)
                return

    if "play music on youtube" in command:
        speak("Which song would you like to listen to?")
        song = listen(timeout=4)
        if song:
            speak(f"Playing {song} on YouTube.")
            query = song.replace(" ", "+")
            webbrowser.open(f"https://www.youtube.com/results?search_query={query}")

            speak("Would you like to download this song?")
            answer = listen(timeout=4)
            if "yes" in answer or "yeah" in answer:
                download_song(song)
            else:
                speak("Okay, not downloading.")
        else:
            speak("Sorry, I didn't catch the song name.")
        return

    if "open notepad" in command:
        speak("Opening Notepad.")
        os.system("notepad")
        return

    if "what is your name" in command:
        speak("My name is Buddy, your virtual assistant.")
        return

    if command in ["exit", "quit", "stop", "shutdown"]:
        speak("Shutting down. Goodbye!")
        exit()

    speak("Sorry, I can't perform that command yet.")

# --- Main Program ---
if __name__ == "__main__":
    greet_user()
    speak("Hello! I am Buddy. Call me by saying 'Hey Buddy' whenever you need me.")

    while True:
        print("Waiting for wake word...")

        # Listen for wake word (faster)
        wake_command = listen(timeout=5)

        if WAKE_WORD in wake_command:
            speak("I'm listening. What can I do for you?")
            user_command = listen(timeout=4)

            if not user_command:
                speak("I didn't catch that. Please try again.")
                continue

            best_match, confidence = process.extractOne(user_command, commands_list)
            print(f"Best match: {best_match} ({confidence}% confidence)")

            if confidence > 60:
                execute_command(best_match)
            else:
                speak("I didn't understand that command. Please try again.")
        else:
            time.sleep(1)  # faster retry
