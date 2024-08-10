import feedparser
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Replace with your podcast RSS feed URL
PODCAST_FEED_URL = "https://fiachrajdavispodcast.podomatic.com/rss2.xml"

# Replace with your email credentials
EMAIL_HOST = "smtp.gmail.com"
EMAIL_PORT = 587
EMAIL_USER = "your_email@gmail.com"
EMAIL_PASS = "your_password"

# Replace with the last known episode link (to avoid duplicate notifications)
LAST_KNOWN_EPISODE = "link_of_last_known_episode"

# List of subscribers to notify
SUBSCRIBERS = [
    "subscriber1@example.com",
    "subscriber2@example.com",
]

def fetch_latest_episode(feed_url):
    """Fetches the latest episode from the podcast RSS feed."""
    feed = feedparser.parse(feed_url)
    latest_episode = feed.entries[0] if feed.entries else None
    return latest_episode

def send_email_notification(episode, subscribers):
    """Sends an email notification to all subscribers."""
    subject = f"New Podcast Episode: {episode.title}"
    body = f"A new episode of your favorite podcast is out!\n\nTitle: {episode.title}\nLink: {episode.link}\n\nEnjoy!"

    for subscriber in subscribers:
        msg = MIMEMultipart()
        msg['From'] = EMAIL_USER
        msg['To'] = subscriber
        msg['Subject'] = subject

        msg.attach(MIMEText(body, 'plain'))

        try:
            with smtplib.SMTP(EMAIL_HOST, EMAIL_PORT) as server:
                server.starttls()
                server.login(EMAIL_USER, EMAIL_PASS)
                server.sendmail(EMAIL_USER, subscriber, msg.as_string())
                print(f"Notification sent to {subscriber}")
        except Exception as e:
            print(f"Failed to send notification to {subscriber}: {str(e)}")

def main():
    latest_episode = fetch_latest_episode(PODCAST_FEED_URL)
    
    if latest_episode and latest_episode.link != LAST_KNOWN_EPISODE:
        send_email_notification(latest_episode, SUBSCRIBERS)
        # Update the last known episode link to avoid re-sending notifications
        global LAST_KNOWN_EPISODE
        LAST_KNOWN_EPISODE = latest_episode.link
    else:
        print("No new episode found.")

if __name__ == "__main__":
    main()
