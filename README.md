# summer_project_with_tkinter
import tkinter as tk
from tkinter import ttk
import pyttsx3
import speech_recognition as sr
from tkinter import messagebox
import subprocess
import boto3
import tkinter.filedialog as filedialog
import requests
from PIL import Image, ImageTk
import os
import re
import cv2
import qrcode
import webbrowser
import tkinter.simpledialog as simpledialog

def do_something():
    messagebox.showinfo("Info", "Team 15 - TechOps")

def system_app():
    system_window = tk.Toplevel(root)
    system_window.title("System Apps")
    system_window.geometry("400x275")
    system_window.configure(bg="lightgray")

    system_label = tk.Label(system_window, text="System Apps", font=("Arial", 20,))
    system_label.pack(pady=10)

    notepad_button = tk.Button(system_window, text="Open Notepad", width=25, fg='Black', command=open_notepad)
    notepad_button.pack(pady=10)

    calculator_button = tk.Button(system_window, text="Open Calculator", width=25, fg='Black', command=open_calculator)
    calculator_button.pack(pady=10)

    system_info_button = tk.Button(system_window, text="System Info", width=25, command=show_system_info)
    system_info_button.pack(pady=10)

    close_button = tk.Button(system_window, text="Close",width=25, command=system_window.destroy)
    close_button.pack(pady=10)

def aws_operations_window():
    aws_window = tk.Toplevel(root)
    aws_window.title("AWS Operations")
    aws_window.geometry("400x350")
    aws_window.configure(bg="lightgray")

    aws_label = tk.Label(aws_window, text="AWS Operations", font=("Arial", 20, "bold"))
    aws_label.pack(pady=10)

    ec2_button = tk.Button(aws_window, text="Create EC2 Instance",width=25, command=open_ec2_instance)
    ec2_button.pack(pady=15)

    s3_button = tk.Button(aws_window, text="Create S3 Bucket",width=25, command=create_s3_bucket)
    s3_button.pack(pady=15)

    s3_button = tk.Button(aws_window, text="Upload to S3",width=25, command=upload_to_s3)
    s3_button.pack(pady=15)

    list_ec2_button = tk.Button(aws_window, text="List EC2 Instances", width=25, command=list_ec2_instances)
    list_ec2_button.pack(pady=15)

    close_button = tk.Button(aws_window, text="Close",width=25, command=aws_window.destroy)
    close_button.pack(pady=10)

def on_exit():
    if messagebox.askyesno("Exit", "Are you sure you want to exit?"):
        root.destroy()

def open_notepad():
    subprocess.Popen("notepad.exe")

def open_calculator():
    subprocess.Popen("calc.exe")

import boto3

def list_ec2_instances():
    try:
        ec2 = boto3.client("ec2")
        response = ec2.describe_instances()
        instances = response["Reservations"]

        if not instances:
            messagebox.showinfo("No EC2 Instances", "No EC2 instances found.")
        else:
            instance_info = "\n".join([f"ID: {instance['Instances'][0]['InstanceId']}, "
                                       f"State: {instance['Instances'][0]['State']['Name']}"
                                       for instance in instances])
            messagebox.showinfo("EC2 Instances", f"List of EC2 instances:\n{instance_info}")
    except Exception as e:
        messagebox.showerror("Error", f"Failed to list EC2 instances: {e}")

def open_ec2_instance():
    response = messagebox.askyesno("AWS EC2 Instance", "Do you want to create an EC2 instance?")
    if response:
        myec2 = boto3.client("ec2")
        response = myec2.run_instances(
            ImageId='ami-0ded8326293d3201b',
            InstanceType='t2.micro',
            MaxCount=1,
            MinCount=1
        )
        print(response)

def create_s3_bucket():
    response = messagebox.askyesno("AWS S3 Bucket", "Do you want to create an S3 bucket?")
    if response:
        s3 = boto3.client('s3')
        s3 = s3.create_bucket(
            Bucket='tejalpg',
            ACL='private',
            CreateBucketConfiguration={
                'LocationConstraint': 'ap-south-1'
            }
        )
        print("Bucket created successfully with the following response:")
        print(s3)
        print("Bucket 'new' was created in the 'ap-south-1' region.")

def upload_to_s3():
    bucket_name = simpledialog.askstring("Upload to S3 Bucket", "Enter the bucket name:")
    if bucket_name:
        file_path = filedialog.askopenfilename(title="Select a file to upload")
        if file_path:
            try:
                s3 = boto3.client("s3")
                file_name = os.path.basename(file_path)
                s3.upload_file(file_path, bucket_name, file_name)
                messagebox.showinfo("Upload Successful", f"File '{file_name}' uploaded to '{bucket_name}'")
            except Exception as e:
                messagebox.showerror("Error", f"Failed to upload file: {e}")

def open_youtube():
    song_name = simpledialog.askstring("Open YouTube", "Enter the name of your favorite song:")
    if song_name:
        search_query = song_name.replace(" ", "+")
        webbrowser.open(f"https://www.youtube.com/results?search_query={search_query}")

def show_system_info():
    response = messagebox.askyesno("System Info", "Want to see ?")
    system_info = subprocess.check_output("systeminfo", shell=True)
    messagebox.showinfo("System Information", system_info)

def google_search():
    search_query = simpledialog.askstring("Google Search", "Enter your search query:")
    if search_query:
        webbrowser.open(f"https://www.google.com/search?q={search_query}")

def capture_video():
    if messagebox.askyesno("Exit", "Want to Capture ?"):
        cap = cv2.VideoCapture(0)
        fourcc = cv2.VideoWriter_fourcc(*'XVID')
        out = cv2.VideoWriter("captured_video.avi", fourcc, 20.0, (640, 480))

    while True:
        ret, frame = cap.read()
        if not ret:
            break
        out.write(frame)

        cv2.imshow("Video", frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    out.release()
    cv2.destroyAllWindows()

    messagebox.showinfo("Video Captured", "Video captured and saved as 'captured_video.avi'")

import requests

def get_weather():
    city = simpledialog.askstring("Weather Update", "Enter city name:")
    if city:
        api_key = "Use-weather-api"
        url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}&units=metric"
        response = requests.get(url)
        weather_data = response.json()
        if weather_data["cod"] == 200:
            weather_info = f"Weather in {city}: {weather_data['weather'][0]['description']}\n" \
                           f"Temperature: {weather_data['main']['temp']}°C\n" \
                           f"Humidity: {weather_data['main']['humidity']}%"
            messagebox.showinfo("Weather Update", weather_info)
        else:
            messagebox.showerror("Error", "Failed to fetch weather data.")

def voice_assistant():
    engine = pyttsx3.init()
    engine.say("How can I assist you?")
    engine.runAndWait()

    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        audio = recognizer.listen(source)

    try:
        recognized_text = recognizer.recognize_google(audio)
        messagebox.showinfo("Voice Assistant", f"You said: {recognized_text}")
    except sr.UnknownValueError:
        messagebox.showerror("Error", "Sorry, I couldn't understand what you said.")
    except sr.RequestError:
        messagebox.showerror("Error", "Sorry, the speech recognition service is currently unavailable.")

def generate_qr_code():
    data = simpledialog.askstring("QR Code Generator", "Enter the text or URL to encode:")
    if data:
        qr = qrcode.QRCode(version=1, box_size=10, border=5)
        qr.add_data(data)
        qr.make(fit=True)

        img = qr.make_image(fill_color="black", back_color="white")
        img.save("qr_code.png")
        img.show()

# Create the main Tkinter window
root = tk.Tk()
root.title("Menu")
root.geometry("580x600")
root.configure(bg="lightblue")
title_font = ("Arial", 24, "italic")
root.iconbitmap(r"C:\Users\sagar\Downloads\Anaconda\logo.png")

# Welcome Text
title_label = tk.Label(root, text="Welcome to GUI Based Menu", font=title_font, fg="blue")
title_label.pack(pady=10)

# Welcome Text
title_label = tk.Label(root, text="Select Options Below", font=("Arial", 18, "italic"), fg="blue")
title_label.pack(pady=10)

notepad_button = tk.Button(root, text="System Apps", width=25, fg='Black', command=system_app)
notepad_button.pack(pady=10)

capture_video_button = tk.Button(root, text="Capture Video", width=25, command=capture_video)
capture_video_button.pack(pady=10)

weather_button = tk.Button(root, text="Weather Update", width=25, command=get_weather)
weather_button.pack(pady=10)

aws_button = tk.Button(root, text="AWS Operations", width=25, command=aws_operations_window)
aws_button.pack(pady=10)

browser_button = tk.Button(root, text="Youtube", width=25, command=open_youtube)
browser_button.pack(pady=10)

google_button = tk.Button(root, text="Google Search", width=25, command=google_search)
google_button.pack(pady=10)

voice_assistant_button = tk.Button(root, text="Voice Assistant", width=25, command=voice_assistant)
voice_assistant_button.pack(pady=10)

qr_code_button = tk.Button(root, text="Generate QR Code", width=25, command=generate_qr_code)
qr_code_button.pack(pady=15)

button = tk.Button(root, text="Exit",width=25, command=on_exit)
button.pack(pady=10)

# Create a menu bar
menu_bar = tk.Menu(root)
root.config(menu=menu_bar)

# Create a file menu
file_menu = tk.Menu(menu_bar, tearoff=0)
file_menu.add_separator()
file_menu.add_command(label="Exit", command=on_exit)
menu_bar.add_cascade(label="File", menu=file_menu)

# Create a help menu
help_menu = tk.Menu(menu_bar, tearoff=0)
help_menu.add_command(label="Team Info", command=do_something)
menu_bar.add_cascade(label="About", menu=help_menu)

# Start the Tkinter event loop
root.mainloop()

