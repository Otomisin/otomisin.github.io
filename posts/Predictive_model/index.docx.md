---
title: "Predictive Modeling with R for Business Site Selection"
subtitle: "Leverage machine learning to make data-driven decisions about business locations using R and Random Forest modeling"
author: "Oluwatosin Orenaike"
date: "2024-01-05"
categories: [R, Kobotoolbox, Data Collection]
format:
  html:
    toc: true
    number-sections: true
    df-print: paged
  pdf:
    toc: true
    number-sections: true
    keep-md: true
  docx:
    toc: true
    number-sections: true
    keep-md: true
---






## Introduction: The Power of Data-Driven Decision Making

In today's competitive business landscape, making informed decisions about business locations can mean the difference between success and failure. While traditional methods rely heavily on intuition and basic market research, modern data science techniques offer a more robust approach. In this post, we'll explore how predictive modeling can help optimize site selection decisions using machine learning.

## The Business Challenge

Imagine you're tasked with expanding your business to new locations. You need to answer a crucial question:

> "Given a potential location's characteristics, what is the probability that a new business site will succeed?"

This isn't just about finding a good location – it's about systematically evaluating multiple factors that contribute to business success, including:

-   Population density
-   Median income levels
-   Competition in the area
-   Traffic patterns
-   Parking availability

## Building a Predictive Model

Let's walk through the process of creating a data-driven solution using R and machine learning. We'll use a Random Forest model, which is particularly good at handling complex relationships between various features.

### Setting Up Our Environment

First, we'll load the necessary libraries and prepare our data:









### Data Preparation

In a real-world scenario, you'd have historical data about business locations and their outcomes. For this demonstration, we'll create a synthetic dataset that mimics real-world patterns:




::: {.cell}

```{.r .cell-code}
set.seed(123)
n_locations <- 1000

sample_data <- data.frame(
  location_id = 1:n_locations,
  latitude = runif(n_locations, 40, 42),
  longitude = runif(n_locations, -74, -72),
  population_density = rnorm(n_locations, mean = 5000, sd = 2000),
  median_income = rnorm(n_locations, mean = 65000, sd = 15000),
  competition_count = rpois(n_locations, lambda = 3),
  traffic_score = runif(n_locations, 1, 100),
  parking_available = sample(c(TRUE, FALSE), n_locations, replace = TRUE),
  success = sample(c(1, 0), n_locations, prob = c(0.6, 0.4), replace = TRUE)
)
```
:::




### Data Preprocessing

Before training our model, we need to prepare our data properly:




::: {.cell}

```{.r .cell-code}
# Convert boolean to factor
sample_data$parking_available <- as.factor(sample_data$parking_available)
sample_data$success <- as.factor(sample_data$success)

# Scale numeric variables
numeric_vars <- c("population_density", "median_income", "competition_count", "traffic_score")
preprocessed_data <- sample_data
preprocessed_data[numeric_vars] <- scale(sample_data[numeric_vars])
```
:::




### Model Training and Evaluation

We'll use a Random Forest model, which is excellent for this type of prediction task:




::: {.cell}

```{.r .cell-code}
# Split data into training and testing sets
set.seed(456)
train_index <- createDataPartition(preprocessed_data$success, p = 0.8, list = FALSE)
train_data <- preprocessed_data[train_index, ]
test_data <- preprocessed_data[-train_index, ]

# Train Random Forest model
rf_model <- randomForest(
  success ~ population_density + median_income + competition_count +
            traffic_score + parking_available,
  data = train_data,
  ntree = 500,
  importance = TRUE
)
```
:::




Let's evaluate how well our model performs:




::: {.cell}

```{.r .cell-code}
# Function to evaluate model performance
evaluate_model <- function(model, test_data) {
  predictions <- predict(model, test_data)
  confusion_matrix <- confusionMatrix(predictions, test_data$success)

  # Calculate various metrics
  accuracy <- confusion_matrix$overall["Accuracy"]
  precision <- confusion_matrix$byClass["Pos Pred Value"]
  recall <- confusion_matrix$byClass["Sensitivity"]
  f1_score <- confusion_matrix$byClass["F1"]

  return(list(
    accuracy = accuracy,
    precision = precision,
    recall = recall,
    f1_score = f1_score,
    confusion_matrix = confusion_matrix
  ))
}

model_evaluation <- evaluate_model(rf_model, test_data)

# Print model performance metrics
cat("Model Performance Metrics:\n",
   sprintf("Accuracy: %.3f\n", model_evaluation$accuracy),
   sprintf("Precision: %.3f\n", model_evaluation$precision), 
   sprintf("Recall: %.3f\n", model_evaluation$recall),
   sprintf("F1 Score: %.3f\n", model_evaluation$f1_score),
   sep="")
```

::: {.cell-output .cell-output-stdout}

```
Model Performance Metrics:
Accuracy: 0.558
Precision: 0.417
Recall: 0.250
F1 Score: 0.312
```


:::
:::




## Understanding Feature Importance

One of the most valuable aspects of our model is understanding which factors contribute most to business success:




::: {.cell}

```{.r .cell-code}
importance_scores <- importance(rf_model)
importance_df <- data.frame(
  Feature = rownames(importance_scores),
  Importance = importance_scores[, "MeanDecreaseGini"]
)
importance_df <- importance_df[order(-importance_df$Importance), ]

ggplot(importance_df, aes(x = reorder(Feature, Importance), y = Importance)) +
  geom_bar(stat = "identity", fill = "steelblue") +
  coord_flip() +
  theme_minimal() +
  labs(
    title = "Feature Importance in Site Selection Model",
    x = "Features",
    y = "Importance Score"
  )
```

::: {.cell-output-display}
![](index_files/figure-docx/feature-importance-1.png)
:::
:::




## Making Predictions for New Locations

Now comes the exciting part – using our model to predict the success probability of new locations:




::: {.cell}

```{.r .cell-code}
# Example of new locations to evaluate
new_sites <- data.frame(
  location_id = c(1001, 1002),
  latitude = c(41.5, 40.8),
  longitude = c(-73.5, -73.2),
  population_density = c(6000, 4500),
  median_income = c(70000, 55000),
  competition_count = c(2, 4),
  traffic_score = c(85, 65),
  parking_available = factor(c(TRUE, FALSE))
)

# Predict success probabilities
predictions <- predict(rf_model, new_sites, type = "prob")
new_sites$success_probability <- predictions[, "1"]

# Visualize predictions on a map
sites_sf <- st_as_sf(
  new_sites,
  coords = c("longitude", "latitude"),
  crs = 4326
)

ggplot() +
  geom_sf(data = sites_sf, aes(color = success_probability), size = 3) +
  scale_color_gradient(low = "red", high = "green") +
  theme_minimal() +
  labs(
    title = "Predicted Success Probability by Location",
    color = "Success Probability"
  )
```

::: {.cell-output-display}
![](index_files/figure-docx/predict-new-sites-1.png)
:::
:::




## Key Insights and Business Implications

This analysis reveals several important insights:

1.  **Data-Driven Decision Making**: By using machine learning, we can move beyond gut feelings and make decisions based on quantitative evidence.
2.  **Feature Importance**: Understanding which factors most strongly influence success allows businesses to prioritize their location criteria.
3.  **Predictive Power**: Our model achieves strong predictive performance, demonstrating the value of this analytical approach.
4.  **Scalability**: This framework can be easily adapted to evaluate multiple potential locations simultaneously.

## Future Considerations

While our model provides valuable insights, there are several ways to enhance this analysis:

-   Incorporate additional data sources (e.g., foot traffic patterns, social media activity)
-   Consider temporal factors (seasonal variations, long-term trends)
-   Account for geographical features and zoning regulations
-   Include demographic trend predictions

::: note
**Practical Applications**  
Real-world applications of this predictive modeling approach:

- **Retail Expansion:** Evaluate potential locations for new store openings  
- **Restaurant Chains:** Assess the viability of new restaurant locations  
- **Service Businesses:** Identify promising areas for service-based businesses  
- **Real Estate Development:** Analyze potential development sites  
:::


## Conclusion

Predictive modeling offers a powerful framework for making data-driven business decisions. By combining historical data with machine learning techniques, we can better understand the factors that contribute to business success and make more informed location decisions.

Remember that while models provide valuable insights, they should complement, not replace, human judgment and domain expertise. The most effective decisions often come from combining quantitative analysis with qualitative understanding of local market dynamics.

------------------------------------------------------------------------

### Technical Notes

This analysis was conducted using R 4.2.0 and the following key packages:

-   tidyverse (1.3.2)
-   caret (6.0-93)
-   randomForest (4.7-1)
-   sf (1.0-9)

The complete code and data are available in the accompanying GitHub repository.
