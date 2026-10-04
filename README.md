# EcoSort: Integrated Waste Management Assistant

An AI assistant that helps residents dispose of waste correctly. Given a **photo** or a **text description** of an item, it predicts the waste category and returns **Metro City recycling instructions** together with the policy documents they are based on.

Built as the summative lab for *Neural Networks and Similar Models* (Moringa School).

## How it works

```
image ──► CNN (MobileNetV2) ─────┐
                                 ├──► category ──► retrieve policies ──► generate instructions ──► answer + sources
text  ──► DistilBERT classifier ─┘     (9 classes)    (bge-small + FAISS)    (fine-tuned flan-t5-base)
```

| Component | Model | What it does |
|---|---|---|
| Image classification | MobileNetV2 (transfer learning, last 80 layers fine-tuned) | Classifies photos into 9 waste categories |
| Text classification | DistilBERT (all layers fine-tuned) | Classifies written descriptions into the same 9 categories |
| Instruction generation (RAG) | bge-small-en-v1.5 + FAISS retrieval, fine-tuned flan-t5-base | Retrieves the relevant policies and writes grounded recycling instructions |
| Integrated assistant | `waste_management_assistant()` | Accepts an image or text, flags low-confidence predictions, returns instructions and sources, records user feedback |

Categories: Cardboard, Food Organics, Glass, Metal, Miscellaneous Trash, Paper, Plastic, Textile Trash, Vegetation.

## Results (approximate, can vary 1 to 2% between runs)

| Component | Result |
|---|---|
| CNN | about 0.89 test accuracy (475 test images) |
| Text classifier | about 0.999 test accuracy; about 0.75 on a challenge set of 36 realistic descriptions (TF-IDF baseline: about 0.64) |
| Instructions | full policy coverage, all content grounded in the retrieved documents, 0 contradictions |
| Full assistant | about 0.90 end-to-end accuracy on test images; low-confidence answers are flagged |

## Repository contents

| File | Description |
|---|---|
| `waste_management_summative.ipynb` | The full notebook with code, outputs and observations for all five parts |
| `README.md` | This file |

## Data

The data is not included in this repository.

- **RealWaste images** (4,752 images, 9 classes): [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/908/realwaste)
- **waste_descriptions.csv** (5,000 generated item descriptions) and **waste_policy_documents.json** (14 generated Metro City policy documents): provided with the course materials

## How to run

The notebook was built in **Google Colab** and reads its data from Google Drive.

1. Upload `realwaste.zip`, `waste_descriptions.csv` and `waste_policy_documents.json` to a folder in your Google Drive.
2. Open the notebook in Colab and set `DRIVE_DIR` in the first code cell to that folder, for example `"/content/drive/MyDrive/YourFolderName"`.
3. Select a GPU: **Runtime > Change runtime type > T4 GPU**.
4. Run **Runtime > Run all**. A full run takes about 40 minutes. Trained models are saved to a `models` folder in the same Drive folder (about 1.5 GB).

Main libraries: TensorFlow/Keras, PyTorch, Hugging Face Transformers, sentence-transformers, FAISS, scikit-learn, textstat.

## Key design decisions

- **Fixed image split:** the provided split leaked 129 images between validation and test. The held-out images are split by file path instead (475 validation, 475 test, no overlap).
- **Conflicting policies:** some documents contradict each other (for example on window glass and plastic bags). The category-specific policy wins, and contradicting lines are removed before indexing.
- **Grounded generation:** the generator is trained to answer from the retrieved documents. When the policy text is edited, the answer follows the edit.
- **Data-driven confidence thresholds:** 0.7 for images and 0.85 for text, chosen from validation and challenge-set results. Below the threshold the assistant asks the user to confirm.

## Limitations

- The text data is generated and templated, so test scores overstate real-world performance.
- All photos have similar grey backgrounds, so real resident photos may score lower.
- The generator mostly copies policy lines rather than writing new explanations.
- Some errors are made with high confidence, and inputs that are not waste items still receive a category.

## Team

- Cyrus Kirui
- Nickson Kipruto

## Acknowledgements

RealWaste dataset from the UCI Machine Learning Repository. Policy documents and item descriptions provided by Moringa School.
