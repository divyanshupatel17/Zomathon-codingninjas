# NextBite — CSAO Rail Recommendation Engine

> **Zomathon 2026** &mdash; Team Pixels &bull; Problem Statement 2: Cart Super Add-On (CSAO) Rail Recommendation System
>
> *Predicting the next perfect bite — AI-powered, context-aware add-on recommendations that complete every meal.*

[![Deploy to GitHub Pages](https://github.com/divyanshupatel17/Zomathon-codingninjas/actions/workflows/deploy.yml/badge.svg)](https://github.com/divyanshupatel17/Zomathon-codingninjas/actions/workflows/deploy.yml)

---

## Project Info

| Field | Details |
|---|---|
| **Project Name** | Zomato CSAO Recommendation System |
| **Hackathon** | Zomathon |
| **Organizer** | BLOCKSEBLOCK |
| **Hackathon URL** | [Event Page](https://blockseblock.com/hackathon_details/0ae4aba5-0028-4f47-b44f-ec9eba01d1ef) |
| **Team Name** | Pixels |
| **Team Lead** | Divyanshu Patel (9301503581) |
| **Team Member** | Varshith Pilli (8978930678) |
| **Problem Statement** | Cart Super Add-On (CSAO) Rail Recommendation System |
| **Submission Deadline** | 2nd March 2026, 2:00 PM |

---

## Live Site

The project is deployed via GitHub Pages:

**[https://divyanshupatel17.github.io/Zomathon-codingninjas/](https://divyanshupatel17.github.io/Zomathon-codingninjas/)**

| Page | URL |
|---|---|
| Project Landing Page | [`/`](https://divyanshupatel17.github.io/Zomathon-codingninjas/) |
| Analytics Dashboard | [`/zomato_csao/dashboard.html`](https://divyanshupatel17.github.io/Zomathon-codingninjas/zomato_csao/dashboard.html) |
| Full PDF-Ready Report | [`/zomato_csao/report_full.html`](https://divyanshupatel17.github.io/Zomathon-codingninjas/zomato_csao/report_full.html) |
| Original HTML Report | [`/zomato_csao/csao_report.html`](https://divyanshupatel17.github.io/Zomathon-codingninjas/zomato_csao/csao_report.html) |
| PDF Submission | [`/docs/Zomato_CSAO_Recommendation_System.pdf`](https://divyanshupatel17.github.io/Zomathon-codingninjas/docs/Zomato_CSAO_Recommendation_System.pdf) |

---

## Problem Statement

How can we build an intelligent recommendation system that suggests relevant add-on items to customers based on their current cart composition, contextual factors, and historical behavior patterns, while maintaining high acceptance rates and customer satisfaction?

## Solution Summary (Hackathon-ready)

Pixels built a complete, deployable CSAO recommendation system that balances accuracy, novelty, and production constraints. Key highlights:

- LightGBM learning-to-rank model (LambdaRank) trained on 92K+ cart events with 40+ engineered features.
- 50-dimensional text embeddings for item cold-start and semantic similarity, enabling immediate relevance for new menu items.
- Real-world performance: AUC 0.743 vs baseline 0.574, projected AOV +18.4%, projected CSAO accept rate 47.3% (baseline 39.1%).
- Low-latency production architecture (Redis feature store + LightGBM inference) achieving P95 ≈ 41ms.
- End-to-end deployment plan: shadow testing, canary rollout, KPI guardrails (C2O, cart abandonment), and monitoring dashboards.

This solution is engineered to be both competitive (strong offline metrics) and safe-to-deploy (operational guardrails and benchmarking), aligning directly with Zomathon evaluation criteria.

### Key Challenges

- **Cart Context Understanding** &mdash; Identifying complementary items and incomplete meal patterns (main dish without beverage, etc.)
- **Contextual Relevance** &mdash; Incorporating time of day, restaurant type, user segment, and geographic context
- **Real-Time Constraint** &mdash; Predictions must be served in under 200-300ms at scale
- **Cold Start Problem** &mdash; Handling new users, restaurants, and items with no prior interaction data
- **Sequential Logic** &mdash; Re-scoring candidates as items are added to the cart (Biryani → Salan → Gulab Jamun → Drinks)

---

## Dataset

Synthetic dataset designed to mimic real-world food delivery patterns across 7 major Indian cities.

| Entity | Count | Key Attributes |
|---|---|---|
| Users | 5,000 | 3 segments, 7 cities, dietary preferences, behavioral history |
| Restaurants | 800 | 10 cuisine types, 4 price bands, chain vs. independent |
| Menu Items | 7,914 | Main, side, dessert, beverage categories |
| Sessions | 10,000 | 5 meal slots, 3 device types, weekend signals |
| Cart Events | 92,680 | 36,846 cart adds + 55,834 CSAO interactions |

**CSAO Acceptance Rate: 39.1%** (21,806 accepted / 55,834 total CSAO events)

### User Distribution
- **Segments:** Budget (49.6%), Occasional (30.4%), Premium (20.0%)
- **Cities:** Mumbai, Bangalore, Hyderabad, Pune, Delhi, Kolkata, Chennai
- **Dietary:** Non-veg (39.8%), Veg (30.8%), Both (29.4%)

### Session Distribution
- **Meal Time:** Dinner (26.7%), Lunch (25.2%), Breakfast (22.7%), Late Night (14.2%), Snack (11.2%)
- **Device:** Mobile (75.3%), Web (20.3%), Tablet (4.5%)

---

## Solution Architecture

### Approach

The CSAO recommendation problem is framed as a **learning-to-rank binary classification** task:
- **Model:** LightGBM with LambdaRank objective
- **Features:** 40+ contextual features across user, restaurant, cart, and temporal dimensions
- **Candidate Generation:** Rule-based pre-filter narrows 7,914 items to 50-100 candidates per request
- **Inference:** Redis-cached pre-computed embeddings + online feature assembly

### Real-Time Pipeline

```
Cart Trigger (2ms)
  --> Feature Fetch from Redis (8ms)
    --> Candidate Generation (5ms)
      --> LightGBM Scoring (18ms)
        --> Ranked Response (8ms)
          = P95 Total: ~41ms (well within 200ms SLA)
```

### Feature Engineering

Key feature groups:
1. **Cart State Features** &mdash; `cart_has_main`, `cart_has_beverage`, `cart_category_diversity`, `meal_complete`
2. **User Features** &mdash; `user_segment`, `csao_acceptance_rate`, `preferred_meal_time`, `veg_preference`
3. **Temporal Features** &mdash; `meal_time`, `is_weekend`, `day_of_week`
4. **Candidate Features** &mdash; `candidate_category`, `actual_attach_rate`, `price_diff_from_cart_avg`
5. **Restaurant Features** &mdash; `primary_cuisine`, `price_band`, `rating`, `popularity`

---

## Model Performance

Evaluation on temporal holdout split:

| Metric | LightGBM Model | Baseline | Improvement |
|---|---|---|---|
| AUC-ROC | 0.743 | 0.574 | +29.4% |
| Precision @ 3 | 0.618 | 0.421 | +46.8% |
| Recall @ 5 | 0.571 | 0.398 | +43.5% |
| NDCG @ 5 | 0.682 | 0.489 | +39.5% |
| NDCG @ 10 | 0.651 | 0.462 | +40.9% |
| P95 Latency | 41ms | 12ms | Within 200ms SLA |

### Top Feature Importances

1. `cart_has_main` &mdash; 9.2%
2. `actual_attach_rate` &mdash; 8.8%
3. `meal_time` &mdash; 8.2%
4. `candidate_category` &mdash; 7.8%
5. `cart_category_diversity` &mdash; 7.3%
6. `user_segment` &mdash; 6.9%
7. `price_diff_from_cart_avg` &mdash; 6.5%

---

## Business Impact Projections

| Metric | Baseline | LightGBM (Projected) | Lift |
|---|---|---|---|
| Average Order Value (AOV) | baseline | +18.4% | +18.4% |
| CSAO Acceptance Rate | 39.1% | 47.3% | +8.2pp |
| CSAO Attach Rate | 27.4% | 38.2% | +10.8pp |
| Cart-to-Order (C2O) Ratio | 2.9 | 3.4 | +17.2% |
| Avg Items per Order | 2.3 | 2.8 | +21.7% |

---

## A/B Testing Framework

| Parameter | Value |
|---|---|
| Control Group | 50% &mdash; Popularity-based baseline |
| Treatment Group | 50% &mdash; LightGBM CSAO model |
| Primary Metric | AOV lift |
| Guardrail Metric | C2O rate (must not decrease) |
| Sample Size | 100,000 sessions minimum |
| Duration | 2 weeks minimum |

---

## Repository Structure

```
.
├── index.html                        # Project landing page
├── README.md                         # This file
├── .github/
│   └── workflows/
│       └── deploy.yml                # GitHub Pages deployment action
├── docs/
│   └── Zomato_CSAO_Recommendation_System.pdf  # Official submission PDF
└── zomato_csao/
    ├── dashboard.html                # Analytics dashboard (Chart.js visualizations)
    ├── csao_report.html              # Full HTML report
    ├── index.html                    # Navigation page
    ├── users.csv                     # 5,000 synthetic users
    ├── restaurants.csv               # 800 restaurants
    ├── items.csv                     # 7,914 menu items
    ├── sessions.csv                  # 10,000 sessions
    ├── cart_events.csv               # 92,680 cart events
    ├── user_features.csv             # Engineered user features
    ├── item_features.csv             # Engineered item features
    ├── restaurant_features.csv       # Engineered restaurant features
    └── csao_features.csv             # Full feature-engineered dataset
```

---

## Submission Details

- **Hackathon:** Zomathon 2026 by BLOCKSEBLOCK in collaboration with Zomato
- **Problem Statement:** Cart Super Add-On (CSAO) Rail Recommendation System (Problem Statement 2)
- **Deadline:** 2nd March 2026, 2:00 PM
- **Submission Format:** Single PDF (included in `/docs/`)
- **Team Lead:** Divyanshu Patel

---

*All data in this repository is synthetic and was created for the purposes of this hackathon. Please refer to the Terms & Conditions accepted during registration regarding confidentiality obligations.*
