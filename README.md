# 🛡️ Phishing & Spam Message Detector

**Classifies spam/phishing-style messages — and explains exactly which signals triggered the flag.**

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat&logo=scikitlearn&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat&logo=streamlit&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green.svg)

---

## An honest framing, up front

No dedicated phishing-email/URL dataset was available for this project — only the SMS Spam Collection (ham vs. spam SMS messages). Rather than mislabel what this is, it's built and documented as a **text-based spam/phishing-style message classifier**. It generalizes reasonably well because phishing and spam share the same manipulation tactics — urgency, prize bait, suspicious links, requests to "verify" something — which is exactly why the tool surfaces *which* signals it detected, not just a verdict.

## Architecture

```
Raw message text (SMS Spam Collection — 5,572 messages, ham/spam)
        │
        ├──► Clean text (lowercase, strip punctuation except $/£/€/!)
        │         │
        │         ▼
        │    TF-IDF vectorizer (unigrams + bigrams, 3,000 features)
        │
        └──► Engineer 8 signal features:
             length, exclamation count, digit count, has_url,
             has_phone_number, urgency_word_count, money_word_count,
             ALL_CAPS_word_count
        │
        ▼
   Combine (TF-IDF sparse matrix + scaled engineered features)
        │
        ▼
   Logistic Regression classifier ──► spam/phishing probability
        │
        ▼
   Streamlit app: risk gauge + "why this was flagged" + awareness tips
```

## Key engineering decisions

- **Compared two model families before picking one.** Naive Bayes got perfect precision but noticeably lower recall; Logistic Regression traded a little precision for better, more balanced recall and a higher ROC-AUC — the right call for a security-relevant tool, where missing real phishing (a false negative) matters more than over-flagging.
- **Added engineered features on top of TF-IDF, not instead of it.** Pure bag-of-words misses structural signals — a message can be phishing-flavored without using any single "spammy" word, just by combining a URL + urgency + a long number.
- **Chose Logistic Regression for interpretability.** Its coefficients directly show which words/signals push a prediction toward spam, which lets the app show a genuine "why this was flagged" list instead of a black-box score.

## Results

| Model | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|
| Multinomial NB (TF-IDF only) | 1.000 | 0.813 | 0.897 | 0.986 |
| Logistic Regression (TF-IDF only) | 0.898 | 0.898 | 0.898 | 0.990 |
| **Logistic Regression + engineered features (deployed)** | **0.991** | **0.898** | **0.943** | **0.992** |

Dataset: 5,572 messages (4,516 ham / 641 spam after deduplication) — `class_weight="balanced"` used to address the imbalance.

## Tech stack

Python · scikit-learn (TfidfVectorizer, MultinomialNB, LogisticRegression) · pandas · NumPy · SciPy · Streamlit

## Project structure

```
train_model.py          Cleans text, engineers features, trains + compares models
app.py                   Streamlit app: message analyzer + model/dataset dashboard
phishing_model.pkl        Final deployed Logistic Regression model
vectorizer.pkl            Fitted TF-IDF vectorizer
engineered_scaler.pkl     Fitted scaler for the 8 engineered features
model_metrics.json        Full metrics, confusion matrix, top trigger words
requirements.txt
```

## Run it locally

```bash
pip install -r requirements.txt
python train_model.py     # regenerates model.pkl, vectorizer.pkl, metrics
streamlit run app.py
```

## License

MIT
