# JARVIS - AI Voice Assistant

An AI-powered voice assistant built with Python that listens to voice 
commands and performs tasks automatically — from searching the web to 
sending WhatsApp messages and emails.

Built by: Mohammad Sadulla Khan & Danish

---

## Features

- Greets user based on time of day (Morning / Afternoon / Evening)
- Voice-controlled Google and YouTube search
- Plays YouTube videos by voice command
- Sends WhatsApp messages via voice
- Sends Emails via voice (Gmail)
- Opens applications — Chrome, Notepad, Camera, Spotify, Calculator, CMD
- Searches Wikipedia and reads out results
- Plays local MP3 music and stops on command
- Gets current time and public IP address
- 95% speech recognition accuracy using Google Speech API

---

## Tech Stack

| Library | Purpose |
|---|---|
| pyttsx3 | Text-to-speech (JARVIS speaks back) |
| SpeechRecognition | Converts voice to text |
| Google Speech API | Powers the recognition engine |
| pywhatkit | WhatsApp messaging & YouTube search |
| wikipedia | Fetches Wikipedia summaries |
| smtplib | Sends emails via Gmail |
| OpenCV (cv2) | Opens and displays webcam |
| pygame | Plays and controls local music |
| requests | Fetches public IP address |
| subprocess | Launches system applications |

---

## How to Run

1. Clone this repository
   
   git clone https://github.com/Sadullak786/JARVIS-AI-Voice-Assistant.git

2. Install dependencies

   pip install pyttsx3 SpeechRecognition opencv-python pygame requests pywhatkit wikipedia

3. Set your Gmail App Password as environment variable

   On Windows CMD:
   set GMAIL_APP_PASSWORD=your_app_password_here

4. Run the assistant

   python jarvis.py

---

## Voice Commands You Can Use

| Say This | What Happens |
|---|---|
| "Open Chrome" | Opens Google Chrome |
| "Open YouTube" | Opens YouTube in browser |
| "Play [song name]" | Plays video on YouTube |
| "Search [topic]" | Searches Google |
| "Wikipedia [topic]" | Reads Wikipedia summary |
| "Send WhatsApp message" | Sends WhatsApp message |
| "Email to" | Sends an email |
| "Play music" | Plays random MP3 from local folder |
| "Stop music" | Stops the music |
| "What is the time" | Tells current time |
| "IP address" | Reads your public IP |
| "Open camera" | Opens webcam |
| "Exit" / "Quit" | Closes JARVIS |

---

## Project Stats

- Speech Recognition Accuracy: 95%
- Tasks Automated: 15+
- Reduction in manual effort: ~60%

---

## Note

This project was built as a collaborative learning project during MCA 
at Noida Institute of Engineering and Technology. 

Codes - 

import pyttsx3  # Text-to-speech conversion
import speech_recognition as sr  # Convert speech to text
import datetime  # Work with date and time
import os  # OS interactions
import subprocess  # Launch applications or commands
import cv2  # OpenCV for camera access
import random  # Random selection
import pygame  # Play and control music
import requests  # HTTP requests
import smtplib  # Send emails
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
import pywhatkit as kit  # WhatsApp messages, YouTube searches
import pywhatkit
import wikipedia
import webbrowser



# Initialize text-to-speech engine
engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)

# Flag to control music playback
is_music_playing = False

# Speak function
def speak(audio, rate=200):
    print(audio)
    engine.say(audio)
    engine.runAndWait()

# Convert voice to text
def takecommand():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        r.pause_threshold = 3
        try:
            audio = r.listen(source, timeout=5, phrase_time_limit=7)
        except sr.WaitTimeoutError:
            print("Listening timed out due to no response")
            return "none"
        try:
            print("Recognizing...")
            query = r.recognize_google(audio, language='en-in')
            print(f"User said: {query}")
        except sr.UnknownValueError:
            speak("I didn't catch that, could you say it again?")
            return "none"
        except sr.RequestError:
            speak("Sorry, I am having trouble accessing the speech recognition service")
            return "none"
        return query

# Greet the user
def wish():
    hour = int(datetime.datetime.now().hour)
    if 0 <= hour <= 12:
        speak("Good Morning, Sir")
    elif 12 < hour < 18:
        speak("Good Afternoon , Sir")
    else:
        speak("Good evening, Sir")

    introduction = ( "Allow me to introduce myself, I am JARVIS,"
                    "...........a virtual artificial intelligence, created by Mohammad Sadullah khan and Danish  "
                    "I am here to assist you with a variety of tasks, 24/7 a day. "
                    "System is now fully operational. How can i assist you today?")
    speak(introduction, rate=150)
def open_youtube():
    speak("opening Youtube")
    webbrowser.open("https://www.youtube.com")

def search_wikipedia(query):
    try:
        speak("Searching Wikipedia...")
        query = query.lower().replace("search", "").replace("wikipedia", "").strip()
        if query:
            results = wikipedia.summary(query, sentences=5)
            speak("According to Wikipedia")
            speak(results)
            print(results)
        else:
            speak("Please specify what you want to search on Wikipedia.")
    except wikipedia.exceptions.DisambiguationError as e:
        speak("There are multiple results for this topic. Please be more specific.")
        print(f"Options: {e.options}")
    except wikipedia.exceptions.PageError:
        speak("Sorry, I couldn't find anything on Wikipedia for that topic.")
    except Exception as e:
        speak(f"An error occurred: {e}")


# Function to get current time
def get_time():
    current_time = datetime.datetime.now().strftime("%I:%M:%S %p")
    speak(f"The current time is {current_time}")
    print(f"The current time is {current_time}")  # This line will print the time in the output section




# Function to search on Google
def google_search(query):
    speak(f"Searching Google for {query}")
    pywhatkit.search(query)

# Function to play YouTube videos
def play_youtube(video):
    speak(f"Playing {video} on YouTube")
    pywhatkit.playonyt(video)


def send_email():
    sender_email = "sadullakhantalkhapur@gmail.com"
    app_password = os.getenv("GMAIL_APP_PASSWORD")  # Use an environment variable for security

    if not app_password:
        speak("Email authentication failed. Please set up an App Password.")
        return

    # Get recipient email by voice
    speak("Whom do you want to send an email to?")
    to = takecommand()

    if to == "none":
        speak("Sorry, I didn't get the recipient's email. Please try again.")
        return

    # Get subject by voice
    speak("What is the subject of your email?")
    subject = takecommand()

    # Get message content by voice
    speak("What message would you like to send?")
    content = takecommand()

    email_body = f"Subject: {subject}\n\n{content}"

    try:
        server = smtplib.SMTP("smtp.gmail.com", 587)
        server.ehlo()
        server.starttls()
        server.login(sender_email, app_password)
        server.sendmail(sender_email, to, email_body)
        server.close()
        speak("Email sent successfully!")
    except Exception as e:
        speak(f"Failed to send email due to {str(e)}")

# Send email
'''def send_email(sender_email, sender_password, recipient, subject, message_body):
    speak("Preparing to send an email")
    try:
        msg = MIMEMultipart()
        msg['From'] = sender_email
        msg['To'] = recipient
        msg['Subject'] = subject
        msg.attach(MIMEText(message_body, 'plain'))
        server = smtplib.SMTP_SSL('smtp.gmail.com', 465)
        server.login(sender_email, sender_password)
        server.sendmail(sender_email, recipient, msg.as_string())
        server.quit()
        speak("Email has been sent successfully")
    except Exception as e:
        speak(f"Sorry, I couldn't send the email. Error: {e}")'''

# Get public IP address
def get_ip_address():
    try:
        ip = requests.get('https://api.ipify.org').text.strip()
        if ip:
            speak(f"Your IP address is {ip}")
        else:
            speak("Sorry, I couldn't retrieve your IP address.")
    except requests.exceptions.RequestException as e:
        speak(f"Failed to retrieve IP address. Error: {e}")

# Open applications
def open_application(query):
    if 'open notepad' in query:
        speak("Opening Notepad")
        subprocess.call("notepad.exe")

    elif 'open google chrome' in query or 'open chrome' in query:
        speak("Opening Google Chrome")
        subprocess.Popen('"C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe"', shell=True)
        subprocess.Popen("start chrome", shell=True)

    elif 'open command prompt' in query or 'open cmd' in query:
        speak("Opening Command Prompt")
        subprocess.Popen("start cmd", shell=True)
    elif 'open spotify' in query:
        speak("Opening Spotify")
        #subprocess.Popen("Spotify.exe", shell=True)
        #import webbrowser
        webbrowser.open("https://open.spotify.com")


    elif 'open youtube' in query:
        open_youtube()

    elif 'open facebook' in query:
        speak("opening facebook")
        webbrowser.open("https://www.facebook.com")

    elif 'open instagram' in query:
        speak("opening instagram")
        webbrowser.open("https://www.instragram.com")

    elif 'open google' in query:
        speak("sir, what should i search on google")
        cm=takecommand().lower()
        webbrowser.open(f"{cm}")



    elif 'open camera' in query:
        speak("Opening Camera")
        cap = cv2.VideoCapture(0)
        if not cap.isOpened():
            speak("Sorry, I couldn't access the camera")
            return
        cv2.namedWindow("Webcam", cv2.WINDOW_NORMAL)
        cv2.resizeWindow("Webcam", 800, 600)
        while True:
            ret, img = cap.read()
            if not ret:
                speak("Failed to capture image from camera")
                break
            cv2.imshow('Webcam', img)
            if cv2.waitKey(1) & 0xFF == 27:
                break
        cap.release()
        cv2.destroyAllWindows()
        speak("Camera closed successfully")
    elif 'open calculator' in query:
        speak("Opening Calculator")
        subprocess.call('calc.exe')
    #else:
        speak("Sorry, I am not able to open that application right now")

    elif 'email to' in query:
        send_email()

# to close any applicatiins
    elif 'close notepad' in query:
        speak("Ok sir, closing notepad")
        os.system("taskkill /f /im notepad.exe")

    else:
        speak("Sorry, I am not able to open that application right now")



# Play music
def play_music():
    global is_music_playing
    music_dir = "A:\\Music"
    songs = os.listdir(music_dir)
    music_files = [song for song in songs if song.endswith('.mp3')]
    if not music_files:
        speak("No music files found in the directory.")
        return
    speak(f"Playing music from {music_dir}")
    song_to_play = random.choice(music_files)
    speak(f"Now playing: {song_to_play}")
    pygame.mixer.init()
    pygame.mixer.music.load(os.path.join(music_dir, song_to_play))
    pygame.mixer.music.play()
    is_music_playing = True

# Stop music
def stop_music():
    global is_music_playing
    if is_music_playing:
        pygame.mixer.music.stop()
        is_music_playing = False
        speak("Music stopped")
    else:
        speak("No music is currently playing")

# Send WhatsApp message
def send_whatsapp_message():
    try:
        speak("Please enter the phone number, including the country code")
        phone_number = input("Enter the phone number (with country code): ")
        speak("What message would you like to send?")
        message = takecommand()
        now = datetime.datetime.now()
        hour = now.hour
        minute = now.minute + 1
        kit.sendwhatmsg(phone_number, message, hour, minute)
        speak(f"Message will be sent to {phone_number} at {hour}:{minute}")
    except Exception as e:
        speak(f"Sorry, I couldn't send the WhatsApp message. Error: {e}")

# Main program
if __name__ == "__main__":
    wish()
    while True:
        query = takecommand().lower()

        if 'open' in query:



        #elif 'open' in query:
            open_application(query)
        elif 'play music' in query:
            play_music()
        elif 'stop music' in query or 'close music' in query:
            stop_music()
        elif 'ip address' in query:
            get_ip_address()
        elif 'send whatsapp message' in query:
            send_whatsapp_message()
        elif 'wikipedia' in query:
            search_wikipedia(query)
        elif "search" in query:
            query = query.replace("search ", "")
            google_search(query)
        elif "play" in query:
            video = query.replace("play ", "")
            play_youtube(video)
        elif "time" in query:
            get_time()  # Directly call get_time() here

        elif 'exit' in query or 'quit' in query:
            speak("Goodbye, Sir")
            break
