
---

## `ecommerce-shipping-analysis-r/README.md` (FULL replacement)

```md
# E-Commerce Shipping Analysis (R)

I’m analysing an e-commerce shipping dataset to understand which operational factors are most associated with the dataset’s binary delivery outcome `Reached.on.Time_Y.N`.

This repo contains the dataset + a fully reproducible R workflow (included below).

---

## Dataset

File in this repo:
- `E-Commerce-Shipping-Data.csv`

Shape:
- 10,999 rows
- 12 columns

Key fields (from the dataset):
- `Warehouse_block`, `Mode_of_Shipment`, `Product_importance`
- `Customer_rating`, `Cost_of_the_Product`, `Prior_purchases`
- `Discount_offered`, `Weight_in_gms`
- `Reached.on.Time_Y.N` (binary outcome)

---

## Reproduce (run exactly as-is)

### Requirements
R + packages: `readr`, `dplyr`, `ggplot2`, `broom`

### Run
Open R/RStudio in the repo root and run:

```r
packages <- c("readr","dplyr","ggplot2","broom")
installed <- rownames(installed.packages())
for (p in packages) if (!p %in% installed) install.packages(p, dependencies = TRUE)

library(readr)
library(dplyr)
library(ggplot2)
library(broom)

dir.create("outputs", showWarnings = FALSE)

df <- read_csv("E-Commerce-Shipping-Data.csv", show_col_types = FALSE)

# Basic checks
writeLines(paste0("Rows: ", nrow(df), " | Cols: ", ncol(df)), "outputs/dataset_shape.txt")
write.csv(summary(df), "outputs/summary.csv")

# Outcome rate (Outcome==1)
outcome_rate <- mean(df$Reached.on.Time_Y.N, na.rm = TRUE)
writeLines(paste0("Reached.on.Time_Y.N==1 rate: ", round(outcome_rate * 100, 2), "%"), "outputs/outcome_rate.txt")

# Segment stats
by_mode <- df %>%
  group_by(Mode_of_Shipment) %>%
  summarise(
    n = n(),
    outcome1_rate = mean(Reached.on.Time_Y.N, na.rm = TRUE),
    avg_rating = mean(Customer_rating, na.rm = TRUE),
    avg_discount = mean(Discount_offered, na.rm = TRUE),
    avg_weight_g = mean(Weight_in_gms, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  arrange(desc(n))

by_warehouse <- df %>%
  group_by(Warehouse_block) %>%
  summarise(
    n = n(),
    outcome1_rate = mean(Reached.on.Time_Y.N, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  arrange(desc(outcome1_rate))

by_importance <- df %>%
  group_by(Product_importance) %>%
  summarise(
    n = n(),
    outcome1_rate = mean(Reached.on.Time_Y.N, na.rm = TRUE),
    avg_discount = mean(Discount_offered, na.rm = TRUE),
    .groups = "drop"
  ) %>%
  arrange(desc(outcome1_rate))

write.csv(by_mode, "outputs/by_mode.csv", row.names = FALSE)
write.csv(by_warehouse, "outputs/by_warehouse.csv", row.names = FALSE)
write.csv(by_importance, "outputs/by_importance.csv", row.names = FALSE)

# Discount difference by outcome value
disc_by_outcome <- df %>%
  group_by(Reached.on.Time_Y.N) %>%
  summarise(
    n = n(),
    avg_discount = mean(Discount_offered, na.rm = TRUE),
    avg_weight_g = mean(Weight_in_gms, na.rm = TRUE),
    avg_cost = mean(Cost_of_the_Product, na.rm = TRUE),
    .groups = "drop"
  )
write.csv(disc_by_outcome, "outputs/discount_by_outcome.csv", row.names = FALSE)

# Simple logistic regression (predict Outcome==1)
# Note: treat Product_importance / Mode_of_Shipment / Warehouse_block as categorical
df2 <- df %>%
  mutate(
    Warehouse_block = as.factor(Warehouse_block),
    Mode_of_Shipment = as.factor(Mode_of_Shipment),
    Product_importance = as.factor(Product_importance),
    Gender = as.factor(Gender)
  ) %>%
  select(Reached.on.Time_Y.N, Warehouse_block, Mode_of_Shipment, Product_importance, Gender,
         Customer_rating, Cost_of_the_Product, Prior_purchases, Discount_offered, Weight_in_gms) %>%
  na.omit()

fit <- glm(
  Reached.on.Time_Y.N ~ Warehouse_block + Mode_of_Shipment + Product_importance + Gender +
    Customer_rating + Cost_of_the_Product + Prior_purchases + Discount_offered + Weight_in_gms,
  data = df2,
  family = binomial()
)

write.csv(broom::tidy(fit), "outputs/logit_coefficients.csv", row.names = FALSE)

# Plots
p1 <- ggplot(df, aes(x = Discount_offered, fill = as.factor(Reached.on.Time_Y.N))) +
  geom_histogram(bins = 30, alpha = 0.7, position = "identity") +
  labs(
    title = "Discount distribution by outcome value",
    x = "Discount offered",
    fill = "Reached.on.Time_Y.N"
  )

p2 <- ggplot(by_mode, aes(x = Mode_of_Shipment, y = outcome1_rate)) +
  geom_col() +
  labs(title = "Outcome==1 rate by shipment mode", x = "Mode of shipment", y = "Outcome==1 rate")

ggsave("outputs/discount_by_outcome.png", p1, width = 8, height = 5)
ggsave("outputs/outcome_rate_by_mode.png", p2, width = 7, height = 5)

message("Done. Outputs written to ./outputs/")
