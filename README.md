import os
import requests
import json
from datetime import datetime

USERNAME = "VAS-SAN"
LAST_RUN_FILE = "data.json"
GISTS_ENDPOINT = f"https://api.github.com/users/{USERNAME}/gists"

def fetch_gists():
    response = requests.get(GISTS_ENDPOINT)
    if response.status_code == 200:
        return response.json()
    else:
        print("Failed to fetch gists.")
        return []

def load_last_run():
    if os.path.exists(LAST_RUN_FILE):
        try:
            with open(LAST_RUN_FILE, "r") as file:
                return json.load(file)
        except (json.JSONDecodeError, FileNotFoundError):
            return {"last_run_time": None, "last_gist_ids": []}
    else:
        return {"last_run_time": None, "last_gist_ids": []}


def save_last_run(last_run_info):
    # Convert datetime object to string
    last_run_info["last_run_time"] = last_run_info["last_run_time"].isoformat() if last_run_info["last_run_time"] else None
    with open(LAST_RUN_FILE, "w") as file:
        json.dump(last_run_info, file)


def list_new_gists(gists, last_run_info):
    last_run_time = last_run_info["last_run_time"]
    last_gist_ids = last_run_info["last_gist_ids"]

    new_gists = []
    for gist in gists:
        gist_id = gist["id"]
        gist_date = datetime.strptime(gist["created_at"], "%Y-%m-%dT%H:%M:%SZ")

        if last_run_time is None or gist_date > last_run_time or gist_id not in last_gist_ids:
            new_gists.append(gist)

    return new_gists

def display_gists(gists):
    if not gists:
        print("No new gists found since last run.")
        return

    print("New gists since last run:")
    for gist in gists:
        print(f"- {gist['html_url']}")

if __name__ == "__main__":
    last_run_info = load_last_run()
    gists = fetch_gists()
    new_gists = list_new_gists(gists, last_run_info)
    display_gists(new_gists)

      last_run_info["last_run_time"] = datetime.utcnow()
    last_run_info["last_gist_ids"] = [gist["id"] for gist in gists]
    save_last_run(last_run_info)
