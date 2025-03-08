# TikTok-Video-Dataset
A Comprehensive Collection of Trending Videos Across Regions and Categories

import requests
import pandas as pd
from datetime import datetime
import time

# API details
API_URL = "https://tiktok-scraper7.p.rapidapi.com/feed/search"
API_KEY = "8e5a008833msh3fc504684ff1899p118ce0jsn795547757c49"  
HEADERS = {
    "X-RapidAPI-Key": API_KEY,
    "X-RapidAPI-Host": "tiktok-scraper7.p.rapidapi.com"
}

# List of regions to fetch data from
REGIONS = ["us"]

# List of keywords (categories) to search
KEYWORDS = ["fyp", "dance", "comedy", "food", "travel", "fashion", "fitness", "gaming", "music", "art", "education", "sports", "technology", "animals", "beauty"]

# Function to fetch TikTok video data
def fetch_tiktok_data(keywords, region, count, cursor):
    querystring = {
        "keywords": keywords,
        "region": region,
        "count": count,
        "cursor": cursor,
        "publish_time": 0,
        "sort_type": 0
    }
    response = requests.get(API_URL, headers=HEADERS, params=querystring)
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error: {response.status_code}, {response.text}")
        return None

# Function to extract relevant fields from the API response
def extract_video_data(data, keyword, region):
    video_data = []
    if data and data.get("code") == 0:  
        for video in data.get("data", {}).get("videos", []):
            video_data.append({
                "aweme_id": video.get("aweme_id"),
                "video_id": video.get("video_id"),
                "region": region,
                "keyword": keyword,
                "title": video.get("title"),
                "cover": video.get("cover"),
                "duration": video.get("duration"),
                "play_url": video.get("play"),
                "wmplay_url": video.get("wmplay"),
                "size": video.get("size"),
                "wm_size": video.get("wm_size"),
                "music_url": video.get("music"),
                "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            })
    return video_data

# Function to create a dataset
def create_dataset(keywords, regions, count, max_cursor):
    all_video_data = []
    for region in regions:
        for keyword in keywords:
            cursor = 0
            while cursor <= max_cursor:
                print(f"Fetching data for region: {region}, keyword: {keyword}, cursor: {cursor}")
                data = fetch_tiktok_data(keyword, region, count, cursor)
                if data:
                    video_data = extract_video_data(data, keyword, region)
                    all_video_data.extend(video_data)
                    if not data.get("data", {}).get("videos", []):
                        break  # Stop if no more videos are returned
                    cursor = data.get("data", {}).get("cursor", cursor + count)  
                else:
                    break
                time.sleep(1)  s
    return pd.DataFrame(all_video_data)

# Example usage
count = 20  # Number of videos per request
max_cursor = 1000  # Maximum cursor value (adjust based on how much data you want)

# Create dataset
all_data = create_dataset(KEYWORDS, REGIONS, count, max_cursor)

# Save dataset to CSV
output_file = "tiktok_large_dataset.csv"
all_data.to_csv(output_file, index=False)
print(f"Dataset saved to {output_file}")
