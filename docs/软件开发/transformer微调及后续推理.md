---
tags: ["transformer", "微调"]
---

# transformer微调及后续推理

## 微调

```python

from transformers import AutoModelForSequenceClassification

device = torch.device("mps")
model = AutoModelForSequenceClassification.from_pretrained("distilbert/distilbert-base-uncased").to(device)
from transformers import TrainingArguments

training_args = TrainingArguments(
    output_dir="path/to/save/folder/",
    learning_rate=2e-5,
    per_device_train_batch_size=8,
    per_device_eval_batch_size=8,
    num_train_epochs=2,
)
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("distilbert/distilbert-base-uncased")

from datasets import load_dataset

dataset = load_dataset("rotten_tomatoes")  # doctest: +IGNORE_RESULT


def tokenize_dataset(dataset):
    return tokenizer(dataset["text"])


dataset = dataset.map(tokenize_dataset, batched=True)
from transformers import DataCollatorWithPadding

data_collator = DataCollatorWithPadding(tokenizer=tokenizer)

from transformers import Trainer

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"].with_format("torch", device="mps"),
    eval_dataset=dataset["test"].with_format("torch", device="mps"),
    tokenizer=tokenizer,
    data_collator=data_collator,
)
trainer.train()
trainer.save_model("path/to/save/folder")


```

## 使用微调模型推理

```python
from transformers import pipeline

device = torch.device("mps") if torch.backends.mps.is_available() else torch.device("cpu")

pipe = pipeline("fill-mask", model="path/to/save/folder", device=device)
print(pipe("The goal of life is [MASK]."))
```
