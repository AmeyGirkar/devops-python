Here’s a list of essential Python programs that a DevOps engineer should know, each covering common use cases like server monitoring, database interaction, AWS S3 operations, sending emails, and process monitoring:

⸻

1. Server Monitoring (CPU, RAM, Disk)

import psutil

def monitor_server():
    print(f"CPU Usage: {psutil.cpu_percent()}%")
    print(f"Memory Usage: {psutil.virtual_memory().percent}%")
    print(f"Disk Usage: {psutil.disk_usage('/').percent}%")

if __name__ == "__main__":
    monitor_server()



⸻

2. Store Monitoring Data to Database (MySQL Example)

import psutil
import mysql.connector

def store_data():
    cpu = psutil.cpu_percent()
    memory = psutil.virtual_memory().percent
    disk = psutil.disk_usage('/').percent

    conn = mysql.connector.connect(
        host='localhost',
        user='youruser',
        password='yourpassword',
        database='devops_db'
    )
    cursor = conn.cursor()
    cursor.execute("INSERT INTO metrics (cpu, memory, disk) VALUES (%s, %s, %s)", (cpu, memory, disk))
    conn.commit()
    cursor.close()
    conn.close()

if __name__ == "__main__":
    store_data()



⸻

3. S3 Bucket Creation (Using boto3)

import boto3

def create_s3_bucket(bucket_name, region='us-east-1'):
    s3 = boto3.client('s3', region_name=region)
    s3.create_bucket(Bucket=bucket_name, CreateBucketConfiguration={'LocationConstraint': region})
    print(f"Bucket {bucket_name} created.")

if __name__ == "__main__":
    create_s3_bucket('your-unique-bucket-name')



⸻

4. Send Email to User (Using SMTP)

import smtplib
from email.mime.text import MIMEText

def send_email(subject, message, to_email):
    from_email = "your_email@example.com"
    password = "your_password"

    msg = MIMEText(message)
    msg['Subject'] = subject
    msg['From'] = from_email
    msg['To'] = to_email

    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(from_email, password)
        server.sendmail(from_email, to_email, msg.as_string())

if __name__ == "__main__":
    send_email("Alert", "Server CPU usage is high", "user@example.com")



⸻

5. Process Monitoring

import psutil

def monitor_process(process_name):
    for proc in psutil.process_iter(['pid', 'name']):
        if process_name.lower() in proc.info['name'].lower():
            print(f"Running: {proc.info}")
            return
    print(f"{process_name} is not running.")

if __name__ == "__main__":
    monitor_process("nginx")



⸻

Would you like a script that combines multiple functionalities into one, or should I help you containerize these with Docker?
