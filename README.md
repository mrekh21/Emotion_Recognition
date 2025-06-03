# Facial Expression Recognition Challenge 

**Kaggle Competition:**  
[Challenges in Representation Learning: Facial Expression Recognition](https://www.kaggle.com/competitions/challenges-in-representation-learning-facial-expression-recognition-challenge)

**Wandb Report:**  
[Wandb Project Link](https://wandb.ai/mrekh21-free-university-of-tbilisi-/emotion_recognition?nw=nwusermrekh21)

---

## პროექტის მიმოხილვა | Project Overview

ამ პროექტის მიზანია ნეირონული ქსელების გამოყენებით ვიზუალური ემოციების ამოცნობა FER2013 მონაცემებზე. გამოყენებულია **PyTorch**, სხვადასხვა **CNN არქიტექტურები**, ექსპერიმენტული მიდგომები და **Wandb** ტრეკინგი.

---

## მონაცემები | Dataset

- Dataset: `icml_face_data.csv`  
- გამოსახულება: **48x48 grayscale**  
- ემოციების კლასები:
  - `0: Angry`
  - `1: Disgust`
  - `2: Fear`
  - `3: Happy`
  - `4: Sad`
  - `5: Surprise`
  - `6: Neutral`

 **Train/Val/Test გაყოფა:**
- `Training`: 80%
- `PublicTest`: 10%
- `PrivateTest`: 10%
- აქ train.csv-დან შემეძლო ვალიდაციისთვის მონაცემების გამოყოფა (იგივე განაწილებით) და მთლიანი test.csv-ის აღება სატესტოდ, მაგრამ რადგან აღწერაში ეწერა რომ ლიდერბორდისთვის PublicTest-ს იყენებდნენ და PrivateTest-ს გამარჯვებულის გამოსავლენად და რადგან submission example-ში მარტო ერთ-ერთი ტესტ სეტის prediction-ები იყო, აღარ მოვაკელი ტრენინგ სეტს (ერთ ერთი კლასი ისედაც ძალიან ცოტა იყო). 

---

## მონაცემთა დამუშავება | Data Preprocessing

- `pixels` ველის გარდაქმნა `(48, 48, 1)` NumPy array-ში
- ნორმალიზაცია: [0, 1] დიაპაზონში

---

## არქიტექტურები და ექსპერიმენტები | Architectures & Experiments

### 1. Simple CNN

- მარტივი კლასი ზოგადი წარმოდგენის შესაქმნელად, როგორ მუშაობდა CNN არქიტექტურა ამ დავალებისთვის
- 2x Conv2D  
- 1x MaxPooling  
- 2x Fully Connected
- კლასის სისწორეში დასარწმუნებლად მცირე მონაცემებზე გავტესტე, overfit-ში წავიდა და 100%-ით მოერგო

**Accuracy:**  
- Train: 67%  
- Val: 52% ➤ **Overfitting**

---

### 2. Configurable CNN (Hyperparameter Tuning)

- ტესტირებული პარამეტრები:
  - Conv Configs: [[(16, 3), (32, 3), (64, 3)], [(32, 3), (64, 3), (128, 3)]]
  - Dropout: 0.1–0.3
  - Normalization: BatchNorm, LayerNorm, None
  - Pooling: Max, Avg, None
  - Batch size: 64, 128
  - Learning Rate: 0.001, 0.01, 0.05
    
- წინა დავალების მსგავსი ზოგადი CNN კლასის არქიტექტურა გამოვიყენე (3 layer)
- კლასის სისწორეში დასარწმუნებლად მცირე მონაცემებზე გავტესტე, overfit-ში წავიდა და 100%-ით მოერგო
- 30 სხვადასხვა ჰიპერპარამეტრების კონფიგურაცია ამოვარჩიე Random Search-ით ჰიპერპარამეტრების სივრციდან
- ტრენინგის დროს გამოვიყენე Early Stopping
- გამოვიყენე `CrossEntropyLoss` და `Adam Optimizer`
- ამ ექსპერიმენტების ბოლოს ამოვარჩიე საუკეთესო მოდელი
  
**Accuracy:**  
- Train: 76%  
- Val: 60% ➤ **Overfitting**

---

### 3. CNN + Residual ბლოკები (Hyperparameter Tuning)

- ResNet სტილის კავშირები და residual ბლოკები დავამატე CNN-ის კლასს, სავარაუდოდ უკეთესი შედეგი უნდა დაედო და ნაკლები overfitting ჰქონოდა 
- აქაც იგივე კონფიგურაციებია და ჰიპერპარამეტრების სივრცე
- გამოვიყენე `CrossEntropyLoss` და `Adam Optimizer`
- ამოვარჩიე 20 კონფიგურაცია ექსპერიმენტებისთვის და შემდეგ ამოვარჩიე საუკეთესო შედეგის მქონე მოდელი

**Accuracy:**  
- Train: 64%  
- Val: 60% ➤ **ნაკლები Overfitting**

---

### 4. კლასის წონების გამოყენება (class weights)

- აქ `CrossEntropyLoss`-ის ნაცვლად გამოყენებულია `CrossEntropyLoss(weight=class_weights)`
- იგივე ორი ექსპერიმენტი თავისი hyperparameter tuning-ით (CNN, CNN with residuals), ამჯერად class weight-ებით
- შედეგებმა არ გააუმჯობესა მოდელის generalization და უარესი შედეგები ჰქოდათ საუკეთესო მოდელებს

---

##  შეფასება | Evaluation Summary

- საბოლოოდ, residual block-ების დამატებამ გააუმჯობესა მოდელების გენერალიზაცია და შეამცირა overfitting
- მიუხედავად იმისა, რომ კლასები იყო დაუბალანსებელი და კლასი 1 (disgust) იყო ძალიან ცოტა, class weight-ების გამოყენებამ არ გააუმჯობესა შედეგები
- საუკეთესო მოდელად ამოვარჩიე მეორე ექსპერიმენტის საუკეთესო მოდელი, ანუ CNN with residuals ექსპერიმენტის საუკეთესო შედეგის მქონე მოდელი

---

## საუკეთესო მოდელი | Final Best Model

- არქიტექტურა: CNN + Residual ბლოკები
- ConvConfigs: [(32, 3), (64, 3), (128, 3)]
- Dropout: 0.3  
- Norm: BatchNorm  
- Pooling: MaxPooling  
- Epochs: 15  
- EarlyStopping: patience=5  
- LR: 0.001  
- Batch size: 64  

**Accuracy:**
- Train: 62.8%  
- Val: 61.4%  
- Test: 61.8%
- აქ საბოლოოდ ნაკლებ ეპოქაზე დატრენინგდა და უკეთესი შედეგი აქვს (ბევრ ეპოქაზე მეტ ნაკლებად ყველა მოდელი overfit-ში მიდიოდა ან/და val_accuracy პლატოვდებოდა)
- ვალიდაციის და ტესტის accuracy თითქმის იგივეა, სატრენინგო ოდნავ მეტია მაგრამ overfit-ში მაინც არაა და კარგი შედეგი აქვს (სამივე დატაში თითქმის ერთნაირი განაწილება იყო ემოციების ისედაც, ამიტომ კარგ მოდელს დიდი რეინჯები არც უნდა ჰქონოდა შედეგებში)
- სხვა მეტრიკები დეტალურად და confusion matrix არის wandb-ის ლოგებში და რეპორტში

---

## Wandb ლოგირება | Wandb Logging

- ყველა ექსპერიმენტი ინდივიდუალურად ლოგირებულია Wandb-ზე და ყველა ექსპერიმენტისთვის არის report დაგენერირებული (5 რეპორტი, მათ შორის საბოლოო მოდელის ტრენინგის და ევალუაციის შედეგები დეტალურად კლასების მიხედვით)
- თითოეული ექსპერიმენტი არის ცალკე `run`

**Bonus Reports:**  
[Emotion Recognition Report](https://wandb.ai/mrekh21-free-university-of-tbilisi-/emotion_recognition/reportlist)
- 5 რეპორტია: საბოლოო მოდელის ტრენინგი + ევალუაცია დეტალურად, 4 ექსპერიმენტის hyperparameter tuning + best model ტრენინგები

---

## რეპოზიტორიის სტრუქტურა | Repository Structure

```bash
emotion-recognition/
├── notebook.ipynb            # მთავარი ექსპერიმენტების notebook (Colab)
├── README.md                 # ეს ფაილი

