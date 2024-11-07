# Python EDA : Top Spotify Songs in 2023

## TABLE OF CONTENTS

- Project Overview
- Summary of Codes
- Visualization
- Summary of Answers
- Insights
- Challenges
- History

## PROJECT OVERVIEW

This project performs an exploratory data analysis (EDA) on Spotify's most-streamed songs of 2023. The goal is to uncover insights into musical trends, top-performing artists, and characteristics that correlate with popularity, using a combination of statistical summaries and visualizations.


## SUMMARY OF CODES

### OVERVIEW OF DATASET

- Import the libraries
```
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
```

- Load CSV file and use shape function to get number of rows and columns. For this part, I used spotifydf as the name
```
spotifydf = pd.read_csv('spotify-2023.csv', encoding = 'ISO-8859-1')

row_count, column_count = spotifydf.shape
```

- Check Data Types
```
print(spotifydf.dtypes)
```

- Handle Missing Values
```
print("Columns with the missing values:")
print(spotifydf.isnull().sum())
```

### BASIC DESCRIPTIVE STATISTICS 

- Calculate the mean, median, and standard deviation for the streams column
```
#declare stream column as s 
s = 'streams'

#get statistical data (mean, median, and standard deviation)
mean_streams = spotifydf[s].mean()
median_streams = spotifydf[s].median()
stdev_streams = spotifydf[s].std()
```

- Visualize the Distribution of Released_Year and Artist_Count
```
# Create the box plot
fig, axes = plt.subplots(1, 2, figsize=(15, 6))

# Boxplot for released_year
axes[0].boxplot(spotifydf['released_year'], vert=False, widths=0.7, patch_artist=True)
axes[0].set_title('Distribution of Released Year')
axes[0].set_xlabel('Year')

# Boxplot for artist_count
axes[1].boxplot(spotifydf['artist_count'], vert=False, widths=0.7, patch_artist=True)
axes[1].set_title('Distribution of Artist Count')
axes[1].set_xlabel('Number of Artists')

# Display the plots
plt.show()
```

- Finding the Outliers
```
#create a function calculates and displays outliers

def count_outliers(s):
    Q1 = spotifydf[s].quantile(0.25)
    Q3 = spotifydf[s].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    outliers = spotifydf[(spotifydf[s] < lower_bound) | (spotifydf[s] > upper_bound)]
```

- Analyze Trends
```
def analyze_trends(column):
    trend_data = spotifydf[column].value_counts().sort_index()
    trend_df = trend_data.reset_index()
    trend_df.columns = [column, 'Count']
    sns.displot(
        trend_df, x=column, weights='Count', kde=True, bins=len(trend_df), color='pink',
        height=8, aspect=1.5)  #adjust its size 
    plt.title('Trend Analysis of {} Over Time'.format(column), fontsize=16)
    plt.xlabel(column, fontsize=14)
    plt.ylabel('Count', fontsize=14)
    plt.xticks(fontsize=12)
    plt.yticks(fontsize=12)
    plt.show()
```

- Analyze Correlations and Generate Heatmap to observe correlations

- Most Streamed Tracks
```

top_streamed_tracks = spotifydf[['track_name', 'artist(s)_name', 'streams']].sort_values( by = 'streams', ascending = False).head(5)
print("Top 5 Most Streamed Tracks: ")

for i, row in top_streamed_tracks.iterrows():
    print(row['track_name'], "by", row['artist(s)_name'], "with a stream count of: ", row['streams'])

Creating the bar graph:

plt.figure(figsize=(12, 8))
plt.barh(top_streamed_tracks['track_name'] + " by " + top_streamed_tracks['artist(s)_name'], top_streamed_tracks['streams'], color='purple')
plt.xlabel('Streams (in Billions)')
plt.title('Top 5 Most Streamed Tracks')
plt.gca().invert_yaxis()
plt.show()
```

- Most Frequent Artist based on number of tracks
```
top_artists = spotifydf['artist(s)_name'].value_counts().head(5)
for artist, count in top_artists.items():
    print(artist + "with a number of tracks of: " + str(count))

#Create bar graph
plt.figure(figsize=(12, 8))
plt.barh(top_artists.index, top_artists.values, color = 'violet')
plt.xlabel('Number of Tracks')
plt.title('Top 5 Most Frequent Artists')
plt.gca().invert_yaxis()  
plt.show()
```

### TEMPORAL TRENDS

- Number of tracks released over time
```
yearly_release = spotifydf['released_year'].value_counts().sort_index()
monthly_release = spotifydf['released_month'].value_counts().sort_index()
print("Trend Analysis of Track Releases Over Time:")
for year, count in yearly_release.items():
    print("Year:", year, "|| Number of Tracks Released:", count)
```

### GENRE AND MUSIC CHARACTERISTICS

- correlation between streams and musical attributes such as bpm, danceability, and energy.

```
attributes = ['streams', 'bpm', 'danceability_%', 'valence_%', 
              'energy_%', 'acousticness_%', 'instrumentalness_%', 'liveness_%', 'speechiness_%']

correlation_matrix = spotifydf[attributes].corr()
plt.figure(figsize=(12, 8))
sns.heatmap(correlation_matrix, annot=True, cmap='YlOrBr', linewidths=0.5)
plt.title('Correlation Heatmap of Streams and Musical Attributes')
plt.show()
```

- correlation of danceability and energy, also valence and acousticness
```
correlation_matrix = spotifydf[attributes].corr()
danceability_energy_corr = correlation_matrix.loc['danceability_%', 'energy_%']
print("Correlation between Danceability and Energy:{:.2f}".format(danceability_energy_corr))
valence_acousticness_corr = correlation_matrix.loc['valence_%', 'acousticness_%']
print("Correlation between Valence and Acousticness {:.2f}".format(valence_acousticness_corr))
```

### PLATFORM POPULARITY

- Platform Popularity Comparison
```
total_sp_playlists = spotifydf['in_spotify_playlists'].sum()
total_sp_charts = spotifydf['in_spotify_charts'].sum()
total_ap_playlists = spotifydf['in_apple_playlists'].sum()

#platform that favored the most popular tracks
platform = ['Spotify Playlists', 'Spotify Charts', 'Apple Playlists']
total = [total_sp_playlists, total_sp_charts, total_ap_playlists]
```

### ADVANCED ANALYSIS

- Determine pattern among tracks with same key with respect to streams
```
filter_data = spotifydf[spotifydf['key'] != '"N/A"']
key_counts = filter_data['key'].value_counts().sort_index()
key_total_streams = filter_data.groupby('key')['streams'].sum()
key_mean_streams = filter_data.groupby('key')['streams'].mean()
```

- Determine pattern among tracks with same mode with respect to streams
```
mode_counts = spotifydf['mode'].value_counts()
mode_total_streams = spotifydf.groupby('mode')['streams'].sum()
mode_mean_streams = spotifydf.groupby('mode')['streams'].mean()
```

- Analyze if certains artists appear more on playlists or charts
```
playlist_columns = ['in_spotify_playlists', 'in_apple_playlists', 'in_deezer_playlists']
chart_columns = ['in_spotify_charts', 'in_apple_charts', 'in_deezer_charts']

artist_playlist_counts = spotifydf.groupby('artist(s)_name')['total_playlist_occurrences'].sum().sort_values(ascending=False).head(10)
artist_chart_counts = spotifydf.groupby('artist(s)_name')['total_chart_occurrences'].sum().sort_values(ascending=False).head(10)
```

## VISUALIZATIONS

- DISTRIBUTION OF RELEASED_YEAR AND ARTIST_COUNT
```
![download](https://github.com/user-attachments/assets/c1119ee7-18e7-4d55-99e9-94a973e9babe)
```

- TREND ANALYSIS OF RELEASED_YEAR OVER TIME
![download](https://github.com/user-attachments/assets/f2259a40-224a-471e-8253-eff0b9ad5bd0)


- TOP 5 MOST STREAMED TRACKS
![download](https://github.com/user-attachments/assets/5da0c467-97ed-4da0-b068-a00107a803da)

- TOP 5 MOST FREQUENT ARTISTS
![download](https://github.com/user-attachments/assets/7c32a6c0-b758-417e-a3cb-b91628ce7627)
  
- TREND OF TRACKS RELEASED PER YEAR
![download](https://github.com/user-attachments/assets/57f63acf-a1be-41a5-a0d6-600365d1d268)

- TRACKS RELEASED PER MONTH
![download](https://github.com/user-attachments/assets/947be524-bb61-4b77-a9fc-6049201e67e3)

- CORRELATION BETWEEN STREAMS AND MUSICAL ATTRIBUTES
![download](https://github.com/user-attachments/assets/c336e698-cded-4ba2-b0b9-63f32d669be7)

- NUMBER OF TRACKS IN DIFFERENT PLATFORMS
![download](https://github.com/user-attachments/assets/29eebd3d-2ead-4b01-a6cd-cd1cf40a9682)

- CORRELATION BETWEEN STREAMS AND PLATFORM APPEARANCES
![download](https://github.com/user-attachments/assets/b3a28707-f6f8-49ad-9f41-1d294932ba7f)

- PLATFORM FAVORABILITY (MOST POPULAR TRACK)
![download](https://github.com/user-attachments/assets/a29c02d5-3e8f-44ad-b2dd-163a87b8a375)

- NUMBER OF SONGS ACCORDING TO KEY
![download](https://github.com/user-attachments/assets/45d6a122-fa95-49d5-9a15-12a288ced0d2)

- TOTAL STREAMS ACCORDING TO KEY
![download](https://github.com/user-attachments/assets/38c777c6-75a5-49f6-9289-df80567a2fe9)

- MEAN STREAMS ACCORDING TO KEY
![download](https://github.com/user-attachments/assets/20c8feda-3dfe-45d5-90ef-320fcc4f558c)

- NUMBER OF SONGS BY MODE
![download](https://github.com/user-attachments/assets/164109ef-edc9-4c4f-b815-5d2ad7f57e95)

- TOTAL STREAMS BY MODE
![download](https://github.com/user-attachments/assets/4415300f-5c88-4f40-bbb2-751ed7640983)

- TOP 10 ARTISTS IN PLAYLISTS ACROSS PLATFORMS (SPOTIFY, APPLE, AND DEEZER)
![download](https://github.com/user-attachments/assets/c76d55f0-3d69-48ba-9b89-54809499fea1)

- TOP 10 ARTISTS IN CHARTS ACROSS PLATFORMS (SPOTIFY, APPLE, AND DEEZER)
![download](https://github.com/user-attachments/assets/5c280516-beb9-4ddb-8c33-ca00d3d2cc7f)


## SUMMARY OF ANSWERS

* GUIDE QUESTIONS

- OVERVIEW OF DATASET
```
How many rows and columns does the dataset contain?
 The dataset contains **953 rows** and **24 columns**

What are the data types of each column? Are there any missing values?

```

- BASIC DESCRIPTIVE STATISTICS
```
What are the mean, median, and standard deviation of the streamscolumn and artist_count?

Are there any noticeable trends or outliers?

```

- TOP PERFORMERS
```
Which track has the highest number of streams? Display the top 5 most streamed tracks.

Who are the top 5 most frequent artists based on the number of tracks in the dataset?

```

- TEMPORAL TRENDS
```
Analyze the trends in the number of tracks released over time. Plot the number of tracks released per year.

Does the number of tracks released per month follow any noticeable patterns? Which month sees the most releases?
```

- GENRE AND MUSIC CHARACTERISTICS
```
Examine the correlation between streams and musical attributes like bpm, danceability_%, and energy_%. Which attributes seem to influence streams the most?

Is there a correlation between danceability_% and energy_%? How about valence_% and acousticness_%?

```
- PLATFORM POPULARITY
```
How do the numbers of tracks in spotify_playlists, deezer_playlists , and apple_playlists compare?

Which platform seems to favor the most popular tracks?

```

- ADVANCED ANALYSIS
```
Based on the streams data, can you identify any patterns among tracks with the same key or mode (Major vs. Minor)?

Do certain genres or artists consistently appear in more playlists or charts? Perform an analysis to compare the most frequently appearing artists in playlists or charts.
```


