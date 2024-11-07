# Python EDA : Top Spotify Songs in 2023

## TABLE OF CONTENTS

- Project Overview
- Summary of Codes
- Visualization
- Summary of Answers


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
![download](https://github.com/user-attachments/assets/c1119ee7-18e7-4d55-99e9-94a973e9babe)

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
  
How many rows and columns does the dataset contain?
> The dataset contains **953 rows** and **24 columns**

What are the data types of each column? Are there any missing values?
> The dataset consists of both **numerical (int64)** and **categorical (object)** data types. Common missing values include numerical attributes like streams and artist count. 


- BASIC DESCRIPTIVE STATISTICS
  
What are the mean, median, and standard deviation of the streamscolumn and artist_count?

> Mean: **513476285.8651212**

> Median: **287278853**

> Standard Deviation: **567873302.7109758**

Are there any noticeable trends or outliers?

![download](https://github.com/user-attachments/assets/b5ae4523-306c-4e9f-a6d6-324a2de0db38)

> Number of Outliers in 'released_year' : **151**

> Number of Outliers in 'artist_count' : **27**

> Trends : "Based on the graph, the most-streamed Spotify songs predominantly belong to the 2020s. In terms of the number of artists, most songs are produced by either one or two artists. Additionally, the year with the highest number of releases is 2022, with 402 occurrences

- TOP PERFORMERS
  
Which track has the highest number of streams? Display the top 5 most streamed tracks.

> The track with the highest number of streams is **Blinding Lights by The Weeknd with a stream count of : 3703895074**

Who are the top 5 most frequent artists based on the number of tracks in the dataset?

> The top 5 most frequent artists are: 

> Taylor Swift with a number of tracks of: **34**

> The Weeknd with a number of tracks of: **21** 

> SZA with a number of tracks of: **19**

> Bad Bunny with a number of tracks of: 19 Harry Styles with a number of tracks of: **17**


- TEMPORAL TRENDS

Analyze the trends in the number of tracks released over time. Plot the number of tracks released per year.

![image](https://github.com/user-attachments/assets/2fe7cea9-14df-4f6b-9ca2-726a290f90f0)

![download](https://github.com/user-attachments/assets/dc257534-0ecf-4246-a649-28ea81021df2)

Does the number of tracks released per month follow any noticeable patterns? Which month sees the most releases?

![download](https://github.com/user-attachments/assets/fb6104b0-d2b3-41f5-9199-b813053d04ef)

> Based on this graph, as the 'Ber' months approach, the number of songs released gradually increases, peaking during the New Year.

> The month with the highest number of releases is **January**, with a total of **134 tracks**. 

- GENRE AND MUSIC CHARACTERISTICS
 
Examine the correlation between streams and musical attributes like bpm, danceability_%, and energy_%. Which attributes seem to influence streams the most?

![download](https://github.com/user-attachments/assets/fa87d409-dc10-4639-b02c-e3e6b9134583)

> Although the correlation is weak due to the low values, the attribute that appears to influence the streams the most is **speechiness**.

Is there a correlation between danceability_% and energy_%? How about valence_% and acousticness_%?

![download](https://github.com/user-attachments/assets/2e143cca-2a70-460f-a0b6-20b31a146a9b)

> Based on the values shown in the heatmap, the answer to both questions is that there is no significant correlation between the variables. This is because the correlation coefficients are very low, specifically below 0.3, indicating a weak or negligible relationship between the attributes.


- PLATFORM POPULARITY

How do the numbers of tracks in spotify_playlists, deezer_playlists , and apple_playlists compare?

![download](https://github.com/user-attachments/assets/7b7cf8a5-d996-4631-8e68-34a6653bc343)

Which platform seems to favor the most popular tracks?

![download](https://github.com/user-attachments/assets/f8465604-56ef-4c1d-82a9-8dcb89d389a7)

- ADVANCED ANALYSIS
  
Based on the streams data, can you identify any patterns among tracks with the same key or mode (Major vs. Minor)?

SAME KEY:

![download](https://github.com/user-attachments/assets/f8f7f699-067f-43ac-b556-05c37bd8f1fa)

![download](https://github.com/user-attachments/assets/7c654dc7-b0c4-457d-b37e-8796da96c458)

![download](https://github.com/user-attachments/assets/874d6e49-91f0-4dd3-ad34-264374a86f7a)

> Based on the results, it can be observed that the most frequently produced tracks are typically in the keys of **C#, G, and F**. C# again stands out with the highest value, indicating that it is indeed the most popular key among listeners. Following C#, the keys of E and D# come next in terms of mean streams, supporting the idea that certain musical keys, like C#, resonate more with audiences and tend to be more popular on streaming platforms.

SAME MODE:

![download](https://github.com/user-attachments/assets/e1c425e9-c8f2-448e-9832-c46b68af54a2)

![download](https://github.com/user-attachments/assets/bdb47077-f132-4e23-b801-6a7210d42597)

> Based on the graphs, more tracks are made in minor. Minor tracks also have the highest stream count.

Do certain genres or artists consistently appear in more playlists or charts? Perform an analysis to compare the most frequently appearing artists in playlists or charts.

![download](https://github.com/user-attachments/assets/360b3451-9057-4826-b502-c0027cdbdba6)

![download](https://github.com/user-attachments/assets/ac2f390c-9f74-48df-a271-35e89ed355b4)

> The graph shows that several artists, including Ed Sheeran and Harry Styles, frequently appear in both playlists and charts. However, Taylor Swift and The Weeknd stand out as the top artists, practically alternating positions across playlists and charts.


Could I **_be_** doing any more EDA? - Chandler Bing (Friends)
