# project-M1-DS2E

---
title: "VietNam Weather Forecast"
author: "thihongnhung.nguyen"
date: "2025-01-10"
output: html_document
---



## R Markdown
## Load required libraries

```{r}
library(tidyverse)
library(lubridate)
library(caret)
library(randomForest)
library(ggplot2)
library(gridExtra)
library(rmarkdown)
library(sendmailR)
library(forecast)
```

## Load the dataset 

```{r}
weather_data <- read.csv("/Users/nguyennhug/Downloads/weather.csv")
head(weather_data)
summary(weather_data)
```

# Checking for missing values

```{r}

sapply(weather_data, function(x) sum(is.na(x)))
```

# Remove outliers in the `rain` column using IQR
```{r}
Q1 <- quantile(weather_data$rain, 0.25, na.rm = T)
Q3 <- quantile(weather_data$rain, 0.75, na.rm = T)
IQR <- Q3 - Q1

lower_bound <- Q1 - 1.5 * IQR
upper_bound <- Q3 + 1.5 * IQR

weather_data <- weather_data %>% filter(rain >= lower_bound & rain <= upper_bound)
```
##  Time Series Forecasting
```{r}

# Subset data for a specific location (e.g., Hanoi)
data_subset <- weather_data %>% filter(province == "Hanoi")

# Arrange data by date
data_subset <- data_subset %>% arrange(date)

# Create a time series object for maximum temperature
temp_ts <- ts(data_subset$max, start = c(year(min(data_subset$date)), month(min(data_subset$date))), 
              frequency = 12)  # Monthly data

# Plot the time series
autoplot(temp_ts) + ggtitle("Maximum Temperature Time Series")

# Check for stationarity
library(tseries)
adf_test <- adf.test(temp_ts)  # Augmented Dickey-Fuller Test
adf_test

# Build an ARIMA model
arima_model <- auto.arima(temp_ts)
summary(arima_model)

# Forecast the next 6 months
forecast_values <- forecast(arima_model, h = 6)

# Plot the forecast
autoplot(forecast_values) + ggtitle("Temperature Forecast for Next 6 Months")

# Accuracy of the model
accuracy(forecast_values)
```

## Data Visualization
``` {r}
# Histogram of temperatures
hist_max <- ggplot(weather_data, aes(x = max)) + geom_histogram(fill = "orange", bins = 30) +
        ggtitle("Maximum Temperature Distribution")

hist_min <- ggplot(weather_data, aes(x = min)) + geom_histogram(fill = "blue", bins = 30) +
        ggtitle("Minimum Temperature Distribution")

grid.arrange(hist_max, hist_min, ncol = 2)
```


## Model Training and Prediction
```{r}
# Preparing data
set.seed(123)
data <- weather_data %>% select(max, min, rain, humidi, wind, cloud, pressure)

# Split data into training and testing
trainIndex <- sample(seq_len(nrow(data)), size = 1000) 
train_data <- data[trainIndex, ]
test_data <- data[-trainIndex, ]


# Train a Random Forest model
rf_model <- randomForest(max ~ ., data = train_data, ntree = 100)
rf_model

# Predictions
predictions <- predict(rf_model, test_data)

# Evaluate accuracy
RMSE <- sqrt(mean((predictions - test_data$max)^2))
cat("RMSE:", RMSE)
```

## Automated Report Generation

```{r}
# Generate an HTML report
rmarkdown::render("Project_13.Rmd",'pdf_document')
```

# Send email with the report

```{r}
from <- "hongnhung991022@gmail.com"
to <- "nhungnth221099@gmail.com"
subject <- "Vietnam weather forecast"
body <- "The weather prediction report is attached."
attachment <- "weather_report.html"

smtp_server <- list(host.name = "smtp.gmail.com", port = 587, user.name = "hongnhung991022@gmail.com", passwd = "VieNh11541115", ssl = TRUE)
sendmail(from, to, subject, body, control = smtp_server)

```
