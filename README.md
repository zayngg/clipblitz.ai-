# clipblitz.ai-
My clip from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import subprocess
import os
from googleapiclient.discovery import build

app = FastAPI()

class YouTubeLink(BaseModel):
    url: str

@app.post("/process")
async def process_video(link: YouTubeLink):
    try:
        # Validate YouTube link
        youtube = build("youtube", "v3", developerKey="YOUR_API_KEY")
        video_id = link.url.split("v=")[1].split("&")[0]
        video_response = youtube.videos().list(part="snippet", id=video_id).execute()
        if not video_response["items"]:
            raise HTTPException(status_code=400, detail="Invalid or private video")
        
        # Download video (using youtube-dl)
        video_file = f"temp_{video_id}.mp4"
        subprocess.run(["youtube-dl", "-o", video_file, link.url], check=True)
        
        # Generate 10 clips (placeholder; add AI later)
        output_dir = f"clips_{video_id}"
        os.makedirs(output_dir, exist_ok=True)
        for i in range(10):
            output_file = f"{output_dir}/clip_{i+1}.mp4"
            subprocess.run([
                "ffmpeg", "-i", video_file, "-ss", str(i*60), "-t", "60",
                "-c:v", "libx264", "-c:a", "aac", output_file
            ], check=True)
        
        os.remove(video_file)
        return {"clips": [f"clip_{i+1}.mp4" for i in range(10)]}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# Run: uvicorn main:app --host 0.0.0.0 --port 8000
