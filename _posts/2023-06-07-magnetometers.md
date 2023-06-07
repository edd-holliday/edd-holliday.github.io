---
layout: post
title:  "Kakovskaya HPP and Magnetometers"
date:   2023-06-07 12:25:44 +0100
categories: geopolitics, signals analysis, python
use_math: true
---

A project building on work of [Richard Cordaro](https://twitter.com/rrichcord) utilising Magnetic Anomaly Detections to discern phenomena such as explosions, earthquakes, landfalls, or rapid glacial movements.

This project expands upon Cordaro's original work, transitioning from Excel to Python. This shift not only automates the process of data collection and transformation but also enhances the visualisation.

There is potential for further development in correlating the data streams from different ground stations. This enhancement would allow for more precise geospatial analysis, rather than just the temporal analysis presented here.

![Idealised Explosion Pattern](/assets/explosion-pattern.jpg)
*Figure 2: Idealised Explosion Pattern*

The methodology involves comparing the collected data to an "Ideal Explosion Pattern" as it would appear on a magnetometer, as demonstrated by Cordaro. This comparison produces a correlation measure between the two. A high correlation suggests that the sensors may have detected an explosion. The data for this pattern, at a 30-second sample rate, is presented in Table 1.1. Table 1.2 demonstrates the author's data at a 15-second sample rate.

#### Table 1.1: Ideal Explosion Pattern (30-second sample rate) - Richard Cordaro

| Time (sec) | Relative Amplitude |
|:----------:|:------------------:|
|     00     |        0.0         |
|     30     |        0.5         |
|     60     |        1.0         |
|     90     |        0.5         |
|    120     |        0.0         |
|    150     |       0.333        |
|    180     |       0.667        |
|    210     |        1.0         |
|    240     |       0.667        |
|    270     |       0.333        |
|    300     |        0.0         |

The table presented by RC the "Ideal Explosion Pattern" at a 30-second sample rate. The 'Time' column represents the time in seconds from the onset of the explosion, while the 'Relative Amplitude' column represents the magnitude of the magnetic anomaly detected. This pattern will serve as a template for correlating the real-world magnetometer data.

#### Table 1.2: Ideal Explosion Pattern (15-second sample rate) - Richard Cordaro

| Time (sec) | Relative Amplitude |
|:----------:|:------------------:|
|     00     |        0.0         |
|     15     |       0.25         |
|     30     |        0.5         |
|     45     |       0.75         |
|     60     |        1.0         |
|     75     |       0.75         |
|     90     |        0.5         |
|    105     |       0.25         |
|    120     |        0.0         |
|    135     |      0.1665        |
|    150     |       0.333        |
|    165     |        0.5         |
|    180     |       0.667        |
|    195     |      0.8325        |
|    210     |        1.0         |
|    225     |      0.8325        |
|    240     |       0.667        |
|    255     |        0.5         |
|    270     |       0.333        |
|    285     |      0.1665        |
|    300     |        0.0         |

As with Table 1.1, this table represents the "Ideal Explosion Pattern", but at a 15-second sample rate. The 'Time' column represents time in seconds from the onset of the explosion, while the 'Relative Amplitude' column represents the magnitude of the magnetic anomaly detected.

The choice of a 5-minute window is not arbitrary but can be adjusted based on the specific requirements of your analysis. A larger window might smooth out the results, reducing the impact of short-lived anomalies, while a smaller window could make the results more sensitive to brief, intense anomalies, such as explosions. The choice of window size is a trade-off between sensitivity and specificity, and should be guided by your understanding of the nature of the data and the phenomena you're investigating.

## 1. Retreive Data

The Intermagnet team has supplied their data onto the HAPI servers, which allows for easy access to the day's data through `hapiclient`. To use it in your environment, install it using `pip install hapiclient`. It might take a little time to figure it out as the documentation for this particular data source seems to be outdated or non-existent.

Here's the python code to fetch the data:

```python
    from hapiclient import hapi, hapitime2datetime

    # Setup variables for the request

    # The dates we are seeking data between.
    start = "2023-06-05T23:00:00"
    stop = "2023-06-06T02:00:00"

    server = "https://imag-data.bgs.ac.uk/GIN_V1/hapi"
    station = 'sua' # Ground station that collected the data
    published_data = "best-avail" # Data from the source, can be raw, preprocessed or processed. This chooses what is most suitably available.
    cadence = 'PT1M' # 1 minute data intervals, there is higher fidelity for a limited number of ground stations
    orientation = 'xyzf' # Orientation of the sensors.

    # This is collected into the dataset that is requested from the server.
    dataset = f"{station.lower()}/{published_data}/{cadence}/{orientation}"

    parameters = "Field_Vector" # Rather than simply magnitude.

    # Making the request
    data, meta = hapi(server, dataset, parameters, start, stop)

    # Convert the data to a DataFrame
    df_time = pd.DataFrame(hapitime2datetime(data['Time']))
    df_vector = pd.DataFrame(data['Field_Vector'])
    df = pd.concat([df_time, df_vector], axis=1)
    df.columns = ['Time', 'Bx', 'By', 'Bz']
    df.set_index('Time', inplace=True)
```

#### Table 2.1: Ground Stations

| Ground Station | Country       | Place         | Latitude | Longitude |
|:--------------:|:-------------:|:-------------:|:--------:|:---------:|
| SUA            | Romania       | Surlari       | 44.68N   | 26.25E    |
| LVV            | Ukraine       | Lviv          | 44.90N   | 23.75E    |
| GCK            | Rep of Serbia | Grocka        | 44.633N  | 20.767E   |
| BEL            | Poland        | Belsk         | 51.836N  | 20.789E   |
| HRB            | Slovakia      | Hurbanovo     | 47.873N  | 18.19E    |
| PAG            | Bulgaria      | Panagjurishte | 42.515N  | 24.177E   |
| THY            | Hungary       | Tihany        | 46.9N    | 17.89E    |

This table provides a selection of ground stations that can be used to retrieve the magnetic field data. Each station is identified by its code, country, place, latitude, and longitude.

Note that the ground stations in Kiev, Ukraine, and Iznik, Turkey are no longer transmitting data.

## 2. Calculate a deviation from the minimum

Following Richard Cordaro's workflow, he subtracts the minimum value from the y data. This adjustment allows for a better comparison with the Idealised Explosion Pattern. The code below accomplishes the same, but using `pandas` gives us the capability to easily process all three cardinal directions simultaneously.

```python
    # Transform DataFrame
    df['Low Bx'] = df['Bx'] - df['Bx'].min()
    df['Low By'] = df['By'] - df['By'].min()
    df['Low Bz'] = df['Bz'] - df['Bz'].min()
```

This block of code calculates the deviation from the minimum for each component of the magnetic field (Bx, By, Bz). The result is a new DataFrame where each field component has been shifted such that its minimum value is now 0. This transformation enhances the visibility of variations in the data and simplifies the comparison with the Ideal Explosion Pattern.

## 3. Magnetic Anomaly Detection and Correlation Analysis

In the next step, we'll define a function that applies a correlation analysis to our data. This involves a sliding window approach, comparing a portion of the collected data to our Idealised Explosion Pattern. It also identifies potential anomalies where the correlation is above a defined sensitivity level.

The Pearson correlation coefficient between two vectors $x$ and $y$ is defined as:

{% raw %}
$$
r = \frac{{\sum {(x_i - \bar{x})(y_i - \bar{y})}}}{{\sqrt{\sum {(x_i - \bar{x})^2}\sum {(y_i - \bar{y})^2}}}}
$$
{% endraw %}

Where:

- $x_i$ and $y_i$ are individual data points in vectors $x$ and $y$,
- $\bar{x}$ and $\bar{y}$ are the means of vectors $x$ and $y$,
- The summations run over the entire length of the vectors.

In the context of our code, x and y are the window of data and the Idealised Explosion Pattern, respectively. This coefficient ranges from -1 to 1, indicating the degree of linear correlation between the vectors. A correlation close to 1 signifies a strong positive correlation. In this case we use absolute values, as the direction of the vectors we are comparing is not important.

```python
from scipy import stats

    # Define a function that calculates the absolute Pearson correlation coefficient 
    # between a data window and the Ideal Explosion Pattern.
    def per_corr(window, pattern):
        return abs(stats.pearsonr(window, pattern)[0])

    # Define a function that identifies potential magnetic anomalies. 
    # It counts the number of correlations above a specified sensitivity level.
    def magenetic_anomaly_detection(window, sensitivity):
        count = window.gt(sensitivity)
        return count.sum()

    # Create a DataFrame with the correlation and anomaly detection results 
    # for each magnetic field component.
    def create_corr_df(df, station, sample_rate='low', sensitivity=0.6):

        all_vectors = []

        pattern = sample_rate_levels[sample_rate]['pattern']

        window_width = len(pattern)
        
        # Apply the correlation analysis and anomaly detection to each component 
        # of the magnetic field data.
        for label, content in df[['Low Bx', 'Low By', 'Low Bz']].items():
            cc = content.rolling(window_width).apply(lambda x: per_corr(x, pattern))
            amp = cc.rolling(window_width).apply(lambda x: magenetic_anomaly_detection(x, sensitivity))
            amp.rename(f'{station}_{label[-2:]}', inplace=True)

            all_vectors.append(amp)
        
        df_all_vectors = pd.concat(all_vectors, axis=1)
        df_all_vectors.name = station

        return df_all_vectors
```

This code constructs a DataFrame, df_all_vectors, containing the results of the correlation analysis and anomaly detection for each component of the magnetic field. This provides a comprehensive picture of potential magnetic anomalies detected by the ground station.

The code also includes the process of setting a minimum threshold for the correlation coefficient. This sensitivity level is a subjective parameter that you may adjust according to your specific needs or the level of confidence you require.

In the function magnetic_anomaly_detection, correlations greater than the specified sensitivity level are identified as potential magnetic anomalies. This function essentially counts the number of such significant correlations within a rolling window of data.

The results are then processed in steps that correspond to the sample size. For each 5-minute interval, the number of significant correlations is counted and this count is assigned to the time at the beginning of that interval. This way, the function generates a time-series of counts of significant correlations, which could be interpreted as potential magnetic anomalies.

## 4. Results

Using the code above you are able to loop through serveral ground stations to collect and analysis their data. Below are several ploteed charts showing this.

Here is an example of a clickable image that opens in a new tab:

<a href="/assets/mag_anomolies_Kakhovska_HPP.png" target="_blank">
  <img src="/assets/mag_anomolies_Kakhovska_HPP.png" alt="Kakhovska HPP Magnetic Anomalies" width="500" height="600">
</a>
<p style="text-align: center;"><i>Figure 1: Magnetic Anomalies at the Kakhovska Hydro Power Plant</i></p>


![Kakhovska HPP Magnetic Anomalies](/assets/mag_anomolies_Kakhovska_HPP.png)
*Figure 1: Magnetic Anomalies at the Kakhovska Hydro Power Plant*

Plotting th echarts this way has allowed us to easily analysis and fuse data from several locations. The East Component (`By`) charts clearly show a short sharp anomly at around  06 June 00:45 UTC. This is around the time that an explosion, be it mines, or something else causing the collapse of the Kakhovska HPP. The anomoly is also detected in the Vertical Direction (Bz).

### See Also

- [Intermagent, Data source](https://intermagnet.org/#services)
- [Twitter Conversation, how it started](https://twitter.com/rrichcord/status/1558455527608287235)
- [Further explanatory notes](https://radoeka.nl/blog/articles/magnetic-anomaly-detection.html)
