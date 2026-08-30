name: Gulf Jobs AI Agent

on:
  workflow_dispatch:

  schedule:
    - cron: "0 6 * * *"
      timezone: "Europe/Istanbul"

  push:
    paths:
      - "jobs.json"
      - "profile.json"

jobs:
  analyze-jobs:
    runs-on: ubuntu-latest

    steps:
      - name: Download project
        uses: actions/checkout@v4

      - name: Fetch Gulf jobs from Apify
        env:
          APIFY_API_TOKEN: ${{ secrets.APIFY_API_TOKEN }}
        run: |
          python - <<'PY'
          import json
          import os
          import urllib.request
          import urllib.error

          actor_url = (
              "https://api.apify.com/v2/actors/"
              "blackfalcondata~naukrigulf-scraper/"
              "run-sync-get-dataset-items"
          )

          searches = [
              ("Chemical Engineer", "Saudi Arabia"),
              ("Chemical Engineer", "United Arab Emirates"),
              ("Chemical Engineer", "Qatar"),
              ("Process Engineer", "Saudi Arabia"),
              ("Process Engineer", "United Arab Emirates"),
              ("Process Engineer", "Qatar"),
              ("Production Engineer", "Saudi Arabia"),
              ("Production Engineer", "United Arab Emirates"),
              ("Production Engineer", "Qatar")
          ]

          all_jobs = []

          for query, location in searches:
              print(f"Searching: {query} - {location}")

              payload = {
                  "mode": "search",
                  "query": query,
                  "location": location,
                  "maxResults": 2,
                  "includeDetails": False,
                  "compact": True,
                  "excludeEmptyFields": True
              }

              request = urllib.request.Request(
                  actor_url,
                  data=json.dumps(payload).encode("utf-8"),
                  method="POST",
                  headers={
                      "Content-Type": "application/json",
                      "Authorization": (
                          "Bearer " + os.environ["APIFY_API_TOKEN"]
                      )
                  }
              )

              try:
                  with urllib.request.urlopen(
                      request, timeout=900
                  ) as response:
                      result = json.loads(
                          response.read().decode("utf-8")
                      )

                  if isinstance(result, list):
                      all_jobs.extend(result)

              except urllib.error.HTTPError as e:
                  print(f"Search failed: {query} - {location}")
                  print(e.read().decode("utf-8", errors="replace"))

          # Remove duplicates
          unique = {}
          for job in all_jobs:
              key = (
                  job.get("url")
                  or job.get("jobUrl")
                  or (
                      str(job.get("title", "")) + "|" +
                      str(job.get("company", "")) + "|" +
                      str(job.get("location", ""))
                  )
              )
              unique[key] = job

          with open("jobs.json", "w", encoding="utf-8") as f:
              json.dump(
                  list(unique.values()),
                  f,
                  ensure_ascii=False,
                  indent=2
              )

          print(f"Total unique jobs: {len(unique)}")
          PY

      - name: Analyze jobs
        run: |
          echo "================================"
          echo "      GULF JOBS AI AGENT"
          echo "================================"

          python - <<'PY'
          import json

          with open("jobs.json", "r", encoding="utf-8") as f:
              jobs = json.load(f)

          with open("profile.json", "r", encoding="utf-8") as f:
              profile = json.load(f)

          skills = [
              x.lower()
              for x in profile.get("skills", [])
          ]

          roles = [
              x.lower()
              for x in profile.get("target_roles", [])
          ]

          scored = []

          for job in jobs:
              title = str(job.get("title", ""))
              description = str(
                  job.get("description")
                  or job.get("descriptionText")
                  or ""
              )

              text = (
                  title + " " + description
              ).lower()

              matched = [
                  skill for skill in skills
                  if skill in text
              ]

              role_match = any(
                  role in title.lower()
                  for role in roles
              )

              score = min(
                  100,
                  len(matched) * 10 +
                  (30 if role_match else 0)
              )

              scored.append(
                  (score, job, matched)
              )

          scored.sort(
              key=lambda x: x[0],
              reverse=True
          )

          print("")
          print("TOP MATCHES")
          print("================================")

          for score, job, matched in scored[:10]:
              print("")
              print("JOB:", job.get("title", "Unknown"))
              print(
                  "COMPANY:",
                  job.get("company", "Unknown")
              )
              print(
                  "LOCATION:",
                  job.get("location", "Unknown")
              )
              print(
                  "MATCH SCORE:",
                  str(score) + "/100"
              )
              print(
                  "MATCHED SKILLS:",
                  ", ".join(matched) if matched else "None"
              )
              print(
                  "URL:",
                  job.get("url")
                  or job.get("jobUrl")
                  or ""
              )
              print("--------------------------------")
          PY
