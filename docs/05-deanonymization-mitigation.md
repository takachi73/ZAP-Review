## Mitigations

1. Temporal: submitReview() -> queue -> cron every 6h publish batch with random shuffle
2. Stylometric: UI plugin "Anonymize my writing" -> rewrites first-person, standardizes LaTeX macros, removes unique n-grams
3. Evaluation: Use existing OpenReview data, train DistilBERT authorship classifier, show top-5 accuracy drops from 65% -> <20% with mitigation

k-anonymity: Each batch must contain at least 20 reviews from distinct PRK_conf