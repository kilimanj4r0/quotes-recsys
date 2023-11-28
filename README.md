# Quotes Content-Based Recommender System using Diary-style Texts

<h3 align="center">
<a href="https://huggingface.co/spaces/batalovme/quotes-recsys"> 🔥 Streamlit Demo </a>|
<a href="./reports/Final Technical Report.pdf.pdf"> 📋 Detailed Technical Report </a>
</h3>

## Team

🧑‍💻 **Vladimir Makharev** 📧 v.makharev@innopolis.university<br>
🧑‍💻 **Artem Batalov** 📧 a.batalov@innopolis.university<br>
🧑‍💻 **Georgii Budnik** 📧 g.budnik@innopolis.university

## Repository structure

- [**notebooks/**](./notebooks) — main directory with step-by-step self-explanatory Jupyter Notebooks
- [data/](./data) — data we collected and saved for solution
- [references/](./references) — IEEE-style references section
- [reports/](./reports) — interim and final reports in PDF 

## Breaking Down the RecSys Problem

### Our Initial Thoughts

1. Find labeled datasets of diary-style texts and quotes, then somehow play with their labels.
2. Fine just datasets of texts that are close to the diary-style domain and quotes datasets (maybe parse from somewhere), then label all of these manually using one multi-label classifier.

#### Found datasets

1. [go_emotions](https://huggingface.co/datasets/go_emotions)
2. [sem_eval_2018_task_1](https://huggingface.co/datasets/sem_eval_2018_task_1)
3. [journal-entries-with-labelled-emotions](https://www.kaggle.com/datasets/madhavmalhotra/journal-entries-with-labelled-emotions)
4. [Diary-Entry-To-Rap](https://huggingface.co/datasets/chaudha7/Diary-Entry-To-Rap)
5. [Quotes Dataset](https://www.kaggle.com/datasets/hiteshsuthar101/quotes-dataset) 

#### Found classifiers from the top [HuggingFace Trending Models](https://huggingface.co/models?sort=trending&search=emotion)

1. [roberta-base-go_emotions](https://huggingface.co/SamLowe/roberta-base-go_emotions) — 27 labels + Neutral label
2. [twitter-roberta-base-emotion-multilabel-latest](https://huggingface.co/cardiffnlp/twitter-roberta-base-emotion-multilabel-latest) — 11 labels (or 4 labels from `tweetnlp` [[1]]())
3. [bert-base-uncased-emotion](https://huggingface.co/bhadresh-savani/bert-base-uncased-emotion) — 6 labels (Ekman emotions)
4. [emotion_text_classifier](https://huggingface.co/michellejieli/emotion_text_classifier) — 6 labels (Ekman emotions)
5. [EmoRoBERTa](https://huggingface.co/arpanghoshal/EmoRoBERTa) — 27 labels + Neutral label


We will experiment with the **first two classifiers**. There are reasons to decline 3, 4, and 5:
- `roberta-base-go_emotions` outperforms `EmoRoBERTa` on the same dataset
-  `bert-base-uncased-emotion` and `emotion_text_classifier` are trained to predict only 6 Ekman emotions, so we consider it as not enough for our task. Also, according to Demszky *et al.* [[2]](https://arxiv.org/pdf/2005.00547.pdf), the 6 emotion categories proposed by Ekman in 1992 are very basic and recent findings in psychology offered a more complex "semantic space" of emotion.

### Goal

The goal is to build a *content-based recommendation system*. By choosing this approach, we will recommend to user $x$ similar to previous items rated highly by $x$.

We need to construct a dataset of user interactions with items. Our setting is:
- **user**: represented by unique id and text (diary-style) + features of the text (emotions)
- **item**: represented by unique id and quote + features of the quote (emotions)
- **interaction**: weight that denotes whether user liked the item or not, i.e., it can be either -1 or 1

Therefore, by combining all things we will get our utility matrix (dataset).

### Preliminary Solution Steps

1. Solve Gathering Ratings problem
    1. Collect samples that represent **user** and **item** (filtering suitable data)
    2. Construct feature vector for each **user** and **item** by using multi-label classifier (choosing somehow appropriate classifier)
    3. Utilize some model that will create a dataset of "reasonable" pairs (diary-style text, quote) so the we can annotate it.
    4. Build a dataset manually (or using external tools) by creating interaction examples based on the dataset of pairs (diary-style text, quote).
    5. Split the dataset into train and test
2. Solve Extrapolating Utilities problem
    1. Make utility matrix dense (probably it will be dense since we have only numerical features and binary interaction weight)
    2. Train different RecSys models
3. Evaluate extrapolation methods
    1. Evaluate (compare) trained models by well-known RecSys metrics

#### Problems to solve

- How to build a user profile? Most probably, it will be a collection of quotes user like and dislike.
- How to avoid overspecialization? User might have multiple quotes that they like

## Footnotes

RecSys models and metrics can be taken from the most recent and new open-source library `RecTools` released by MTS [[3]](https://github.com/MobileTeleSystems/RecTools). These findings were inspired by Habr article about `RecTools` [[4]](https://habr.com/ru/articles/773126/) and lecture about Recommendation Systems (on the PMLDL course).