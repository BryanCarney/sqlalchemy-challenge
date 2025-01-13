# sqlalchemy-challenge

# Module 10 Challenge files

**.csv Files for Analysis**

[hawaii_measurements.csv](https://github.com/BryanCarney/sqlalchemy-challenge/blob/main/Resources/hawaii_measurements.csv)

[hawaii_stations.csv](https://github.com/BryanCarney/sqlalchemy-challenge/blob/main/Resources/hawaii_stations.csv)

**SQL Lite file**

[hawaii.sqlite](https://github.com/BryanCarney/sqlalchemy-challenge/blob/main/Resources/hawaii.sqlite)

**Coding Files**

[app.py](https://github.com/BryanCarney/sqlalchemy-challenge/blob/main/app.py)

[climate_starter.ipynb](https://github.com/BryanCarney/sqlalchemy-challenge/blob/main/climate_starter.ipynb)

# Instructions

Congratulations! You've decided to treat yourself to a long holiday vacation in Honolulu, Hawaii. To help with your trip planning, you decide to do a climate analysis about the area. 

# Data Analysis

# Part 1: Analyze and Explore the Climate Data

**Initial Setup**

Import Dependencies

    %matplotlib inline
    from matplotlib import style
    style.use('fivethirtyeight')
    import matplotlib.pyplot as plt
    
    import numpy as np
    import pandas as pd
    import datetime as dt

**Reflect Tables into SQLAlchemy ORM**

Python SQL toolkit and Object Relational Mapper

    import sqlalchemy
    from sqlalchemy.ext.automap import automap_base
    from sqlalchemy.orm import Session
    from sqlalchemy import create_engine, func

Create engine to hawaii.sqlite

    engine = create_engine("sqlite:///Resources/hawaii.sqlite") 

Reflect an existing database into a new model

    Base = automap_base()

Reflect the tables

    Base.prepare(engine, reflect=True)

View all of the classes that automap found

    Base.classes.keys()

![image](https://github.com/user-attachments/assets/468e0057-b839-47e5-b631-6abaabfb9cc3)

Save references to each table

    Measurement = Base.classes.measurement
    Station = Base.classes.station

Create our session (link) from Python to the DB

    session = Session(engine)

# Exploratory Precipitation Analysis

Find the most recent date in the data set.

    most_recent_date = session.query(func.max(Measurement.date)).all()
    most_recent_date

![image](https://github.com/user-attachments/assets/538211d7-7e6a-4ec0-88be-a3b98ce4bbf1)

**Design a query to retrieve the last 12 months of precipitation data and plot the results.**

Starting from the most recent data point in the database. 
    
    most_recent_date = session.query(func.max(Measurement.date)).scalar()
    most_recent_date = pd.to_datetime(most_recent_date)
    
Calculate the date one year prior and ensure it is in the correct format

    one_year_ago = most_recent_date - pd.Timedelta(days=365)
    one_year_ago_str = one_year_ago.strftime('%Y-%m-%d')
    one_year_ago_str

Perform a query to retrieve the data and precipitation scores

    precipitation_data = session.query(Measurement.date, Measurement.prcp) \
        .filter(Measurement.date >= one_year_ago_str) \
        .order_by(Measurement.date) \
        .all()

Save the query results as a Pandas DataFrame. Explicitly set the column names

    precipitation_df = pd.DataFrame(precipitation_data, columns=["Date", "Precipitation"])
    precipitation_df.head()

![image](https://github.com/user-attachments/assets/a5255d2e-57fc-414e-8eae-f42be1a63b3f)

Sort the dataframe by date

    precipitation_df['Date'] = pd.to_datetime(precipitation_df['Date']).dt.date
    precipitation_df = precipitation_df.sort_values(by='Date')
    precipitation_df

![image](https://github.com/user-attachments/assets/278e232a-7821-4870-a7b1-f2eb2535b664)

Use Pandas Plotting with Matplotlib to plot the data

    ax = precipitation_df.plot(x='Date', y='Precipitation', kind='bar', figsize=(10, 6), color='blue',width=10.0)
    plt.title('Precipitation Data for the Last Year')
    plt.xlabel('Date')
    plt.ylabel('Precipitation (inches)')

Show only every 200th label as too many dates were reflecting, this was to limit the amount of dates displaying in x axis to fit

    n = 200
    ax.set_xticks(ax.get_xticks()[::n])
    plt.show()

![image](https://github.com/user-attachments/assets/d9af6fd7-4663-4fc2-8098-eb9b100ee030)

Use Pandas to calculate the summary statistics for the precipitation data
    
    precipitation_summary = precipitation_df['Precipitation'].describe()
    print(precipitation_summary)

![image](https://github.com/user-attachments/assets/2f9d74f7-7b58-46db-8441-90be9882ba63)

# Exploratory Station Analysis

Design a query to calculate the total number of stations in the dataset

    total_stations = session.query(func.count(Station.station)).all()
    total_stations

![image](https://github.com/user-attachments/assets/5f99c9dd-e2ca-478e-875c-be484954ece1)

Design a query to find the most active stations (i.e. which stations have the most rows?)
List the stations and their counts in descending order.

    active_stations = session.query(Measurement.station, func.count(Measurement.station)) \
        .group_by(Measurement.station) \
        .order_by(func.count(Measurement.station).desc()) \
        .all()
    
    active_stations

![image](https://github.com/user-attachments/assets/95e9fac6-dd64-4bae-803a-8364044861bf)

Using the most active station id from the previous query, calculate the lowest, highest, and average temperature.

    station_id = 'USC00519281'
    
    temperature_stats = session.query(
        func.min(Measurement.tobs),  
        func.max(Measurement.tobs),  
        func.avg(Measurement.tobs)  
    ).filter(Measurement.station == station_id).all()
    
    min_temp, max_temp, avg_temp = temperature_stats[0]
    print(f"Station: {station_id}")
    print(f"Lowest Temperature: {min_temp}")
    print(f"Highest Temperature: {max_temp}")
    print(f"Average Temperature: {avg_temp}")

![image](https://github.com/user-attachments/assets/05bb99b6-45fa-443a-ac2a-a294e9632282)

Using the most active station id
Query the last 12 months of temperature observation data for this station

    station_id = 'USC00519281'
    most_recent_date = session.query(func.max(Measurement.date)).all()[0][0]
    most_recent_date = pd.to_datetime(most_recent_date)

Calculate the date from one year ago

    one_year_ago = most_recent_date - pd.Timedelta(days=365)

Display the temperature observations for the yearly period

    temperature_data = session.query(Measurement.tobs) \
        .filter(Measurement.station == station_id) \
        .filter(Measurement.date >= one_year_ago.strftime('%Y-%m-%d')) \
        .all()

Converted the data to a Pandas DataFrame so it can be plotted

    temperature_df = pd.DataFrame(temperature_data, columns=['Temperature'])
    temperature_df.head()

![image](https://github.com/user-attachments/assets/b163895d-69d3-4273-b40c-883fac73b16f)

Plot the results as a histogram

    plt.figure(figsize=(10, 6))
    plt.hist(temperature_df['Temperature'], bins=12, color='blue', label='TOBS')
    plt.xlabel('Temperature (°F)')
    plt.ylabel('Frequency')
    plt.legend()
    plt.tight_layout()
    plt.show()

![image](https://github.com/user-attachments/assets/c0f73ac2-055d-494f-9fa5-98c76c024073)

Close Session

    session.close()

# Part 2: Design Your Climate App

Import the dependencies.

    from flask import Flask, jsonify
    from sqlalchemy import create_engine, func
    from sqlalchemy.orm import Session
    from sqlalchemy.ext.automap import automap_base
    import datetime as dt
    import pandas as pd

Database Setup

    engine = create_engine("sqlite:///Resources/hawaii.sqlite")

Reflect an existing database into a new model
    
    Base = automap_base()
    Base.prepare(autoload_with=engine)

Reflect the tables

    Measurement = Base.classes.measurement
    Station = Base.classes.station

Create our session (link) from Python to the DB

    session = Session(engine)

Flask Setup

    app = Flask(__name__)  

# Flask Routes

    @app.route('/')
    def home():
        return (
            '''
            Welcome to the Hawaii Climate API! Available Routes:
            /api/v1.0/precipitation
            /api/v1.0/stations
            /api/v1.0/tobs
            /api/v1.0/<start>
            /api/v1.0/<start>/<end>
            '''
        )

![image](https://github.com/user-attachments/assets/05b33067-e448-48ad-b267-701cff90cfaa)


# Route for precipitation data for the year period

    @app.route('/api/v1.0/precipitation')
    def precipitation():

Get the most recent date in the dataset

        most_recent_date = session.query(func.max(Measurement.date)).all()[0][0]
        most_recent_date = pd.to_datetime(most_recent_date)
    
Calculate the date a for a whole year

        one_year_ago = most_recent_date - pd.Timedelta(days=365)
    
Query the last 12 months of precipitation data

        precipitation_data = session.query(Measurement.date, Measurement.prcp) \
            .filter(Measurement.date >= one_year_ago.strftime('%Y-%m-%d')) \
            .all()
    
        precipitation_summary = {date: prcp for date, prcp in precipitation_data}
        return jsonify(precipitation_summary)

![image](https://github.com/user-attachments/assets/946315f6-5723-47bf-845f-2f34106d2d53)

# Route for stations

    @app.route('/api/v1.0/stations')
    def stations():

Query all stations

        stations_data = session.query(Station.station, Station.name).all()
        stations_list = [{"station": station, "name": name} for station, name in stations_data]
        return jsonify(stations_list)

![image](https://github.com/user-attachments/assets/0ee615dc-1d2a-4cb3-9466-46ecf19577ae)

# Route for temperature observations (tobs) for the most active station in the last year

    @app.route('/api/v1.0/tobs')
    def tobs():

Get the most recent date in the dataset

        most_recent_date = session.query(func.max(Measurement.date)).all()[0][0]
        most_recent_date = pd.to_datetime(most_recent_date)
        one_year_ago = most_recent_date - pd.Timedelta(days=365)
    
Query the dates and temperature observations of the most-active station for the previous year of data.

        most_active_station = session.query(Measurement.station, func.count(Measurement.station)) \
            .group_by(Measurement.station) \
            .order_by(func.count(Measurement.station).desc()).first()[0]
        
        tobs_data = session.query(Measurement.date, Measurement.tobs) \
            .filter(Measurement.station == most_active_station) \
            .filter(Measurement.date >= one_year_ago.strftime('%Y-%m-%d')) \
            .all()
    
Convert query results to a list of dictionaries

        tobs_list = [{"date": date, "temperature": tobs} for date, tobs in tobs_data]    
        return jsonify(tobs_list)

![image](https://github.com/user-attachments/assets/504d16f2-148f-4063-9ec8-d7b7f38056f8)

# Route for temperature statistics (min, avg, max) for a given start date

    @app.route('/api/v1.0/<start>')
    def start_stats(start):
        try:

Check if the start date exists in the dataset.  If not prompt user with a message that the date falls outside of the defined parameters.

            earliest_date, latest_date = session.query(
                func.min(Measurement.date),
                func.max(Measurement.date)
            ).first()
    
            if not (earliest_date <= start <= latest_date):
                return jsonify({
                    "error": f"Date out of range. Please use a date between {earliest_date} and {latest_date}."
                }), 404

![image](https://github.com/user-attachments/assets/3e72aa61-8819-4298-9add-68b1364cb9fe)
    
Query to calculate TMIN, TAVG, and TMAX from the start date to the end of the dataset

            stats = session.query(
                func.min(Measurement.tobs).label("TMIN"),
                func.avg(Measurement.tobs).label("TAVG"),
                func.max(Measurement.tobs).label("TMAX")
            ).filter(Measurement.date >= start).all()
            tmin, tavg, tmax = stats[0]
    
Return the statistics as a dictionary

            return jsonify({
                "Start Date": start,
                "TMIN": tmin,
                "TAVG": tavg,
                "TMAX": tmax
            })
    
        except Exception as e:
            return jsonify({"error": str(e)}), 500

![image](https://github.com/user-attachments/assets/4db3a61f-c3d1-43e1-bf97-cf8dbd21d9ec)

# Route for temperature statistics (min, avg, max) for a given date range (start and end)

@app.route('/api/v1.0/<start>/<end>')
def start_end_stats(start, end):
    try:

Check if the start and end dates exist in the dataset range. If not prompt user with a message that the date falls outside of the defined parameters.

        earliest_date, latest_date = session.query(
            func.min(Measurement.date),
            func.max(Measurement.date)
        ).first()

        if not (earliest_date <= start <= latest_date):
            return jsonify({
                "error": f"Start date out of range. Please use a date between {earliest_date} and {latest_date}."
            }), 404

        if not (earliest_date <= end <= latest_date):
            return jsonify({
                "error": f"End date out of range. Please use a date between {earliest_date} and {latest_date}."
            }), 404

        if start > end:
            return jsonify({
                "error": "Start date must be earlier than or equal to the end date."
            }), 400

![image](https://github.com/user-attachments/assets/a0715a52-ce97-4150-879c-bc47aa52e38e)

![image](https://github.com/user-attachments/assets/677a8c71-4d74-4b30-9bb3-84013d3d2d55)


Query to calculate TMIN, TAVG, and TMAX for the date range

        stats = session.query(
            func.min(Measurement.tobs).label("TMIN"),
            func.avg(Measurement.tobs).label("TAVG"),
            func.max(Measurement.tobs).label("TMAX")
        ).filter(Measurement.date >= start).filter(Measurement.date <= end).all()
        tmin, tavg, tmax = stats[0]

Return the statistics as a dictionary

        return jsonify({
            "Start Date": start,
            "End Date": end,
            "TMIN": tmin,
            "TAVG": tavg,
            "TMAX": tmax
        })

    except Exception as e:
        return jsonify({"error": str(e)}), 500

![image](https://github.com/user-attachments/assets/16b7f3d0-7817-4da1-b083-c30187d1b133)

    if __name__ == '__main__':
        app.run(debug=True)
