# TaskBuilder: Consent Reasoning Pipeline with Ambiguity Detection

This project includes two components:
1. A **mobile app** built with Vue.js for structured data collection.
2. A **Python reasoning pipeline** for scoring user input clarity and mapping it to fuzzy logic for rule-based decision-making.

---

## Structure

- `Root` – Source code for the Vue-based mobile data collection interface.
- `BERT_FuzzyDetection_Consent.ipynb` – Python notebook for reasoning and visualization.
- `consent_session_with_comments.xml` – Example XML file exported from the app.
- `README.md` – This documentation.


---

## 1. Vue.js App Setup

The mobile app was developed using Vue 2 with uni-app syntax

### Prerequisites
- Node.js 14+ (recommended)
- Vue CLI or HBuilderX IDE

### Installation & Run

 If using HBuilderX, import the app folder and run in emulator or physical device.

### Output

The app allows users to:
- Enter structured consent-related tasks
- Score each input
- Export the data as XML (used in Python notebook)

---

## 2. Python Reasoning Pipeline

### Installation


```bash
pip install transformers torch matplotlib pandas beautifulsoup4 seaborn
```

### How to Run

```bash
jupyter notebook
```

Open `BERT_FuzzyDetection_Consent.ipynb` and run all cells.

Make sure `consent_session_with_comments.xml` is present in the same directory.

---

## Model Used

This project uses the [Slomb/Ambig_Question](https://huggingface.co/Slomb/Ambig_Question) transformer model to detect semantic ambiguity in user comments.

---

## Example Output

| Input Comment                | Clarity Score | Fuzzy μ |
|-----------------------------|----------------|----------|
| “I’m almost 18”             | 1.7            | 0.66     |
| “Somewhere in Missouri”     | 1.9            | 0.62     |
| “Not feeling right emotionally” | 2.0        | 0.60     |

Visualizations include bar charts and heatmaps for interpretability.

---

