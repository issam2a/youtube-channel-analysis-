importing libs 

```python
from googleapiclient.discovery import build 
import pandas as pd 
from IPython.display import JSON
```

getting the youtube api key :

```python
api_key = 'AIzaSyAsjQiw04Cww4NTEmTMUKA22UXylW1oxQY'
```

we need the channel ID we can get it from here 

[[FREE] YouTube Channel ID Finder - TunePocket](https://www.tunepocket.com/youtube-channel-id-finder/#channle-id-finder-form)

```python
channel_ids =['UCDybamfye5An6p-j1t2YMsg' ,
           #other chanel ids 
            ]
```

this part is from the reference of the youtube API documentation 

[Channels: list  |  YouTube Data API  |  Google for Developers](https://developers.google.com/youtube/v3/docs/channels/list)

```python
api_service_name = "youtube"
api_version = "v3"

# Get credentials and create an API client

youtube = build(
        api_service_name, api_version, developerKey=api_key)

```

This function `get_channel_stats()` is designed to retrieve **YouTube channel statistics** using the **YouTube Data API v3** and return the results in a **Pandas DataFrame**. Let’s break it down step by step:

```python
def get_channel_stats (youtube , channel_ids):
    all_data = []
    request = youtube.channels().list(
        part="snippet,contentDetails,statistics",
        id= ','.join(channel_ids) 
    )
    response = request.execute()
    for item in response['items']:
        data = {
            'channel_name' : item['snippet']['title'],
            'subscribers': item['statistics']['subscriberCount'],
            'views' : item['statistics']['viewCount'],
            'totalViews': item['statistics']['videoCount'],
            'playListId' : item['contentDetails']['relatedPlaylists']['uploads'],
        
        }
    all_data.append(data)
    return (pd.DataFrame(all_data))
```

### 🔧 Function Signature

```python
def get_channel_stats(youtube, channel_ids):

```

- `youtube`: This is an instance of the YouTube API client (from `googleapiclient.discovery.build()`).
- `channel_ids`: A list of YouTube channel IDs you want to get stats for.

---

### 📦 Step 1: Initialize Storage

```python
all_data = []

```

- This list will store the data (as dictionaries) for each channel.

---

### 🔍 Step 2: Make API Request

```python
request = youtube.channels().list(
    part="snippet,contentDetails,statistics",
    id= ','.join(channel_ids)
)

```

- The API call asks for three sections of data:
    - `snippet`: General info (e.g., title, description)
    - `contentDetails`: Playlists and uploads info
    - `statistics`: Subscribers, views, etc.
- `id=','.join(channel_ids)`: Joins the list of channel IDs into a single comma-separated string.

---

### 📡 Step 3: Execute the API Call

```python
response = request.execute()

```

- This sends the request to the YouTube API and stores the JSON response.

---

### 🔁 Step 4: Loop Over Results

```python
for item in response['items']:

```

- Iterates through each channel's data.

---

### 📋 Step 5: Extract Relevant Fields

```python
data = {
    'channel_name' : item['snippet']['title'],
    'subscribers': item['statistics']['subscriberCount'],
    'views' : item['statistics']['viewCount'],
    'totalVideos': item['statistics']['videoCount'],
    'playListId' : item['contentDetails']['relatedPlaylists']['uploads'],
}

```

- Creates a dictionary with selected fields:
    - `channel_name`: Channel’s title
    - `subscribers`: Subscriber count
    - `views`: Total channel views
    - `totalViews`: Total number of videos
    - `playListId`: The ID for the playlist containing all uploaded videos

---

### 📥 Step 6: Store It

```python

all_data.append(data)

```

- Adds the current channel’s stats to the `all_data` list.

---

### 🧾 Step 7: Return as DataFrame

```python

return pd.DataFrame(all_data)

```

- Converts the list of dictionaries into a Pandas DataFrame for easy analysis.

calling the function 

```python
channel_stats = get_channel_stats(youtube,channel_ids)
channel_stats
```

|  | **channel_name** | **subscribers** | **views** | **totalVideos** | **playListId** |
| --- | --- | --- | --- | --- | --- |
| **0** | Mo Chen | 168000 | 6593513 | 252 | UUDybamfye5An6p-j1t2YMsg |

## Now we are going to use the playlist ids to get all of the videos ids in the channel

from the youtube api reference we go the the list method and customize the request part 

[PlaylistItems: list  |  YouTube Data API  |  Google for Developers](https://developers.google.com/youtube/v3/docs/playlistItems/list?apix_params=%7B%22part%22%3A%5B%22snippet%2CcontentDetails%22%5D%2C%22playlistId%22%3A%22UUDybamfye5An6p-j1t2YMsg%22%7D)

![image.png](attachment:24432047-8830-491b-99ce-e48a0e01d91b:image.png)

and we copy this code :

```python
playlist_id= 'UUDybamfye5An6p-j1t2YMsg'
request = youtube.playlistItems().list(
        part="snippet,contentDetails",
        playlistId=playlist_id
    )
response = request.execute()
```

![image.png](attachment:f49714bd-04cc-45ad-ace7-d79d4ac8c210:image.png)

Now we create a function To retrieve **all video IDs** from a specific **YouTube playlist** 

```python
playlist_id= 'UUDybamfye5An6p-j1t2YMsg'
video_ids = []
def get_video_id (youtube , playlist_id):
request = youtube.playlistItems().list(
part="snippet,contentDetails",
playlistId=playlist_id
)
response = request.execute()
for item in response['items']:
video_ids.append(item['contentDetails']['videoId'])
return video_ids

```

```python
video_ids = get_video_id(youtube,playlist_id)

vidwo_ids
```

the output :
`['YJn7Zicrg_0', 'jrZ8eXn_rzg', 'DfumGOJlVic', 'fJZ_CXPgLLo', 'R2ZVWOGMvsQ']`

we notes that we only have 5 items (video_ids) cause it is the default 

so we can Add a parameter in the request (maxResults = 50 ) 

```python
playlist_id= 'UUDybamfye5An6p-j1t2YMsg'
video_ids = []
def get_video_id (youtube , playlist_id):
request = youtube.playlistItems().list(
part="snippet,contentDetails",
playlistId=playlist_id,
maxResults  = 50
)
response = request.execute()

for item in response['items']:
    video_ids.append(item['contentDetails']['videoId'])

next_page_token=response.get('nextPageToken')

while next_page_token is not None :
    request = youtube.playlistItems().list(
    part="snippet,contentDetails",
    playlistId=playlist_id,
        maxResults =50 ,
        pageToken = next_page_token
    )
    response=request.execute()
    for item in response['items']:
        video_ids.append(item['contentDetails']['videoId'])
    next_page_token = response.get('nextPageToken')

return video_ids
```

- **Function name**: `get_video_id`
- **Parameters**:
    - `youtube`: An instance of the YouTube Data API client (created using the Google API client).
    - `playlist_id`: The ID of the YouTube playlist from which you want to extract video IDs.
- **Returns**: A list of all video IDs in the given playlist.

### Step-by-step Explanation:

### 1. **Initialize an empty list to hold video IDs**

```python

video_ids = []

```

- This list will store all the video IDs found in the playlist.

---

### 2. **First API request to get the first page of playlist items**

```python

request = youtube.playlistItems().list(
    part="snippet,contentDetails",
    playlistId=playlist_id,
    maxResults=50
)
response = request.execute()

```

- `youtube.playlistItems().list(...)`: Creates a request to fetch videos in the playlist.
- `part="snippet,contentDetails"`: Tells the API to return both the `snippet` and `contentDetails` for each video.
    - `snippet` includes title, description, etc.
    - `contentDetails` includes the `videoId`.
- `playlistId=playlist_id`: Specifies the target playlist.
- `maxResults=50`: Fetches up to 50 items (this is the API's **maximum** per request).
- `response = request.execute()`: Executes the request and returns the JSON response.

---

### 3. **Extract video IDs from the first page of results**

```python
for item in response['items']:
    video_ids.append(item['contentDetails']['videoId'])

```

- Iterates over each item (i.e., each video) in the `items` list of the response.
- Extracts the `videoId` from the `contentDetails` part and appends it to `video_ids`.

---

### 4. **Check if there is a next page**

```python
next_page_token = response.get('nextPageToken')

```

- If the playlist has more than 50 videos, YouTube provides a `nextPageToken`.
- This token is used to get the next batch of results.

---

### 5. **Fetch and process subsequent pages (if any)**

```python
while next_page_token is not None:
    request = youtube.playlistItems().list(
        part="snippet,contentDetails",
        playlistId=playlist_id,
        maxResults=50,
        pageToken=next_page_token
    )
    response = request.execute()

```

- Keeps looping while `next_page_token` exists.
- Makes a new request including `pageToken=next_page_token` to get the next set of results.

---

### 6. **Extract video IDs from subsequent pages**

```python
    for item in response['items']:
        video_ids.append(item['contentDetails']['videoId'])
    next_page_token = response.get('nextPageToken')

```

- Same as before: loop over results and extract `videoId`.
- Update `next_page_token` to check if there’s another page.

---

### 7. **Return the final list of video IDs**

```python
return video_ids

```

- After all pages are processed, return the complete list of video IDs.

---

### ✅ Summary of What This Function Does:

1. Connects to YouTube's API.
2. Fetches up to 50 videos per API call from the specified playlist.
3. Continues fetching the next pages using `nextPageToken` until all videos are retrieved.
4. Returns a complete list of all video IDs in the playlist.

now we want to extract the videos statistics based on the video_ids list that we created 

we go the references page of youtube documentation —> videos 

![image.png](attachment:8c16e1c0-8dca-4787-b344-83342dbcdf53:image.png)

```python

def get_video_details(youtube , video_ids):
    all_video_info = []

    for i in range(0 ,len(video_ids) , 50 ):
        request = youtube.videos().list(
            part="snippet,contentDetails,statistics",
            id=','.join(video_ids[i:i+50]) 
        )
        response = request.execute()
    
        for video in response['items']:
            stats_to_keep = {'snippet' : ['channelTitle', 'title' ,'description','tags','publishedAt'],
                             'statistics' : ['viewCount', 'likeCount', 'favoriteCount' , 'commentCount'],
                             'contentDetails':['duration' , 'definition', 'caption']
                            }
            video_info = {}
            video_info['video_id'] = video['id']
        
            for k in stats_to_keep.keys():
                for v in stats_to_keep[k]:
                    try:
                        video_info[v] = video[k][v]
                    except:
                        video_info[v] = None
            all_video_info.append(video_info)            
    return pd.DataFrame(all_video_info)
```

## 🔍 Function Purpose:

The `get_video_details` function retrieves detailed information about **multiple YouTube videos** using their video IDs. It:

- Sends API requests in **batches of 50 video IDs** (which is the max allowed per request by YouTube Data API v3),
- Extracts selected data from the API response,
- Stores that data in a list of dictionaries,
- Returns the result as a Pandas DataFrame.

## 🧠 Step-by-step Explanation:

```python
def get_video_details(youtube , video_ids):
    all_video_info = []

```

- Initializes an empty list to store information for each video.

This list will hold dictionaries, each representing data for one video.

### 🔁 Step 2: Loop through video IDs in batches of 50

```python
for i in range(0, len(video_ids), 50):
```

- The YouTube Data API limits you to **50 video IDs per request**, so you send requests in **chunks**.
- `range(0, len(video_ids), 50)` creates start indices: 0, 50, 100, etc.

---

### 📡 Step 3: Make the API request

```python
    request = youtube.videos().list(
        part="snippet,contentDetails,statistics",
        id=','.join(video_ids[i:i+50])
    )
    response = request.execute()

```

- `youtube.videos().list(...)` prepares the request to get details about up to 50 videos at once.
- `part=` specifies which sections of each video’s data you want:
    - `"snippet"`: General metadata like title, description, etc.
    - `"contentDetails"`: Technical details like duration, resolution.
    - `"statistics"`: View count, like count, etc.
- `id=','.join(video_ids[i:i+50])` joins the current batch of video IDs into a comma-separated string.

---

### 🧱 Step 4: Define what fields you want to extract

```python
    stats_to_keep = {
        'snippet': ['channelTitle', 'title', 'description', 'tags', 'publishedAt'],
        'statistics': ['viewCount', 'likeCount', 'favoriteCount', 'commentCount'],
        'contentDetails': ['duration', 'definition', 'caption']
    }

```

- You want to **keep only specific fields** from the response.
- These are grouped by the API response structure: `snippet`, `statistics`, `contentDetails`.

---

### 🔁 Step 5: Extract data for each video in the response

```python
    for video in response['items']:

```

- Each item in `response['items']` corresponds to one video and contains nested dictionaries for `snippet`, `statistics`, and `contentDetails`.

---

### 🧾 Step 6: Create a dictionary for one video's info

```python
        video_info = {}
        video_info['video_id'] = video['id']

```

- Start a dictionary to store data for this video.
- Add the `video_id`.

---

### 🔍 Step 7: Extract only the desired fields

```python
        for k in stats_to_keep.keys():
            for v in stats_to_keep[k]:
                try:
                    video_info[v] = video[k][v]
                except:
                    video_info[v] = None

```

- `k` will be `'snippet'`, `'statistics'`, and `'contentDetails'`.
- `v` is each field we want under those keys.
- For example: `video['snippet']['title']`, `video['statistics']['viewCount']`, etc.
- `try/except`: If a field doesn’t exist (e.g., no `tags` or `likeCount` hidden), it sets the value as `None`.

---

### 📥 Step 8: Add the video’s data to the list

```python
        all_video_info.append(video_info)

```

- After collecting all needed fields, add this video's data to the main list.

---

### 🧮 Step 9: Return the results as a DataFrame

```python
return pd.DataFrame(all_video_info)

```

- Convert the list of dictionaries into a **Pandas DataFrame** for easy analysis and export.

output :

|  | **video_id** | **channelTitle** | **title** | **description** | **tags** | **publishedAt** | **viewCount** | **likeCount** | **favoriteCount** | **commentCount** | **duration** | **definition** | **caption** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **0** | YJn7Zicrg_0 | Mo Chen | Why Data Storytelling is the #1 Skill Employer... | Check out Adverity here: https://info.adverity... | None | 2025-05-21T14:00:49Z | 6914 | 384 | 0 | 37 | PT11M45S | hd | false |
| **1** | jrZ8eXn_rzg | Mo Chen | How to upload data in Google BigQuery for FREE | I'M HERE TO HELP YOU LEARN DATA SKILLS\n🌐 My w... | None | 2025-05-19T13:40:46Z | 1137 | 75 | 0 | 2 | PT7M48S | hd | false |
| **2** | DfumGOJlVic | Mo Chen | Data Analyst vs Data Product Manager | DATA PORTFOLIO & RESUME: https://mochen.info/ | None | 2025-05-09T14:00:19Z | 3197 | 187 | 0 | 4 | PT59S | hd | false |
| **3** | fJZ_CXPgLLo | Mo Chen | Top paying data jobs and how to get ready for ... | DATA ANALYST\nComplete Data Analyst in Python ... | None | 2025-05-08T14:01:04Z | 7916 | 336 | 0 | 23 | PT11M13S | hd | false |
| **4** | R2ZVWOGMvsQ | Mo Chen | Data Analyst vs Data Steward | DATA PORTFOLIO & RESUME: https://mochen.info/ | None | 2025-05-08T14:00:45Z | 1950 | 96 | 0 | 3 | PT56S | hd | false |
| **...** | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| **248** | YY0-DydcfXs | Mo Chen | Was learning EXCEL WORTH IT? | DATA PORTFOLIO & RESUME: https://mochen.info/\... | None | 2023-01-21T10:00:26Z | 4458 | 192 | 0 | 11 | PT2M53S | hd | false |
| **249** | WkTJCm03Bwk | Mo Chen | DATA ANALYST WORK FROM HOME DESK SETUP | DATA PORTFOLIO & RESUME: https://mochen.info/\... | None | 2023-01-15T10:00:20Z | 61331 | 1585 | 0 | 66 | PT6M56S | hd | false |
| **250** | n0vqkuRyMvQ | Mo Chen | Data Analyst Explains When to Use VLOOKUP vs X... | DATA PORTFOLIO & RESUME: https://mochen.info/\... | None | 2023-01-14T10:00:35Z | 88313 | 1888 | 0 | 79 | PT19M26S | hd | false |
| **251** | 07Fj3vEA-rY | Mo Chen | CONVERT PDF TABLES TO EXCEL TABLES | DATA PORTFOLIO & RESUME: https://mochen.info/\... | None | 2023-01-07T08:42:49Z | 19311 | 457 | 0 | 35 | PT3M18S | hd | false |
| **252** | aXB0fL-pdl0 | Mo Chen | Get any STOCK DATA you want using EXCEL ONLY |... | DATA PORTFOLIO & RESUME: https://mochen.info/\... | None | 2023-01-07T08:42:00Z | 48470 | 747 | 0 | 51 | PT5M48S | hd | false |
