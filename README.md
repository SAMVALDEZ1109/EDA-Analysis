# Python EDA : Top Spotify Songs in 2023

## TABLE OF CONTENTS

- Summary of Codes
- Answers for Questions
- Insights
- Challenges
- History

## SUMMARY OF CODES

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

- Using dtype function to print datatype of each column
```
print(spotifydf.dtypes)
```
